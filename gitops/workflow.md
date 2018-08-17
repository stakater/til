# GitOps workflow

The workflow will comprise of 2 steps:

1. Jenkins Pipeline
2. Flux Sync

## Jenkins Pipeline

An app's Git Repo will be setup on jenkins for PR's and master branch. Pipeline will follow the steps below:

### For CI (PR's)

Each PR creation and commit will trigger a jenkins build for that PR and do the following:

- Fetch Dependencies (If required)
- Run test cases
- Run linting tools (If required)
- Build the app
- Generate a docker image
- Push image to Dockerhub
- Notify the result on PR and Slack

### For CD (master)

For every PR merge, the following steps will be performs in jenkins build flow:

- Fetch Dependencies (If required)
- Run test cases
- Run linting tools (If required)
- Build the app
- Generate a new version (Based on git tags and / or version file)
- Generate a docker image with the new version
- Push image to Dockerhub
- Generate Manifests from chart
- Validate Manifests via [kubeval](https://github.com/garethr/kubeval)
- Commit generated manifests to repo
- Push new chart to chart repo
- Push new chart to flux repos that need that chart
- Notify the result on JIRA Ticket and Slack

## Flux Sync

Now that a new version of an app has been released on the app's repo, we can setup flux to sync that app on our cluster.

### Pre-Requisites and Limitations

- Tiller (the server component of Helm) should be running in the cluster. The Helm operator alpha provided with flux will only deal with an unauthenticated Tiller installation (the default).
- The Helm operator alpha only deals with charts in a single git repository, which you supply to it as a command-line argument, along with a parent directory for charts within that repository. In the resources, charts are given relative to the parent directory.
- It does not support helm secrets
- 1 instance of flux can handle only 1 git repository. So if you need to sync more than 1 repository, you need to have separate instances of flux running for each repository
- It only supports local charts

### Creating Flux Repo

Create a new GitHub repository with the following directory structure:

```text
├── charts
│   └── chart1
│       ├── Chart.yaml
│       ├── README.md
│       ├── templates
│       └── values.yaml
├── namespaces
│   ├── tools.yaml
│   └── cp.yaml
└── releases
    |── helm
    |   ├── tools
    |   |   ├── chart1release.yaml
    |   |   └── chart2release.yaml
    |   └── cp
    |       ├── chart1release.yaml
    |       └── chart2release.yaml
    └── manifests
        ├── tools
        |   ├── app1-deployment.yaml
        |   └── app1-service.yaml
        └── cp
            ├── app2-deployment.yaml
            └── app2-service.yaml
```

***Note:*** The directory structure above is not mandatory but it makes things clear about which resources are where.

#### `charts` directory

This will contain all the helm charts that you need in order to deploy your applications

#### `namespaces` directory

Any namespaces that you need for your applications must be inside this folder

#### `releases` directory

It will basically enclose the helm releases and vanilla manifests in their respective folders i.e., `helm` for helm releases and `manifests` for vanilla manifests

### FluxHelmRelease CRD

The helm releases mentioned above in the directory structure are created using a Custom Resource Definition `FluxHelmRelease` provided by Flux Helm Operator. Following is an example which you can use to create a helm release:

```yaml
apiVersion: helm.integrations.flux.weave.works/v1alpha2
kind: FluxHelmRelease
metadata:
  name: chart1-tools
  namespace: tools
  labels:
    chart: chart1
spec:
  chartGitPath: chart1
  releaseName: chart1-tools
  values: # overrides
    image: stakater/helloworld:latest
```

Flux Helm release fields:

- The `name` is mandatory and needs to follow k8s naming conventions
- The `namespace` is optional. It determines where the resource, and thereby the chart, are created
- If the `releaseName` field is not provided, the relVease name used for Helm will be `$namespace-$name`
- The label `chart` is mandatory, and should match the directory containing the chart
- `chartGitPath` is the directory containing a chart, given relative to the charts path (provided to the helm-operator)
- `values` are user customizations of default parameter values from the chart itself

### Install Weave Flux

Once you've created a git repository with the structure above, add the Weave Flux chart repo:

```bash
helm repo add weaveworks https://weaveworks.github.io/flux
```

Now you can install Weave Flux and its Helm Operator by specifying your repository URL:

```bash
helm install --name flux \
--set rbac.create=true \
--set helmOperator.create=true \
--set git.url=ssh://git@github.com/stefanprodan/gitops-helm \ # your git repository ssh url
--set git.chartsPath=charts \
--namespace flux \
weaveworks/flux
```

The Flux Helm operator provides an extension to Weave Flux that automates Helm Chart releases for it. A Chart release is described through a Kubernetes custom resource named `FluxHelmRelease` as described above. The Flux daemon synchronizes these resources from git to the cluster, and the Flux Helm operator makes sure Helm charts are released as specified in the resources.

Also note that Flux Helm Operator works with Kubernetes 1.9 or newer but I've tested it on 1.8.7 as well and it works fine.

At startup Flux generates a SSH key and logs the public key. Find the SSH public key with:

```bash
kubectl -n flux logs deployment/flux | grep identity.pub | cut -d '"' -f2
```

In order to sync your cluster state with Git you need to copy the public key and create a deploy key with write access on your GitHub repository.

Open GitHub, navigate to your flux git repo, go to ***Setting > Deploy keys*** click on ***Add deploy key***, check ***Allow write access***, paste the Flux public key and click ***Add key***

Once Flux has confirmed access to the repository, it will start deploying the workloads of your flux git repo. After a while you will be able to see the Helm releases listed like so:

```bash
helm list
```

Now whenever you make changes to your git repo, the changes will be synced by Flux on the go