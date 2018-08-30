# Running Helm & Tiller

To run helm, tiller must be installed on your cluster. Typically, tiller needs a Service Account which has cluster-admin role so that tiller can create/update/delete resources in the cluster. Following is the Service Account and ClusterRoleBinding for it.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

Create above resources and now run `helm init --service-account tiller` and it will deploy tiller in `kube-system` namespace.

Usually, tiller is installed in kube-system namespace but we can change that as well. In above command, we can add a flag `--tiller-namespace` and specify the namespace where we want tiller to be deployed, but the corresponding Service Account also needs to be created in that namespace. So e.g. to deploy tiller in `test` namespace, create the above service account in `test` namespace and to deploy tiller, the command becomes `helm init --service-account tiller --tiller-namespace test`.

And for any future command of helm, we would need to define `--tiller-namespace` in every command like for listing releases, `helm ls --tiller-namespace test` or installing release `helm upgrade --install <release-name> <chart-name> --namespace <namespace-of-apps> --tiller-namespace test`.


## Single tiller

We can create a single tiller in `kube-system` namespace, and once initialize your helm and then you can deploy any charts or releases.

## Multiple tillers

Similary, we can create multiple tillers in multiple namespaces, but we would have to create multiple service accounts and clusterrolebindings as above for each namespace. This can be helpful when you want strict isolation in your namespaces/projects and each team is working on a single project and don't want it to mess with other team's resources. However, there are pros and cons of this approach.

### Pros

- Each namespace will have its own releases and a member of one namespace would not be able to create / delete releases of other teams or even list them. e.g. If I list `helm ls --tiller-namespace tiller-test1`, I would only be able to see releases of this specific tiller and not any other.

### Cons

- Everytime when running any command, you would have to explicitly mention the tiller-namespace in every command, and if you miss that, which is likely, it can create in other tiller which can cause unexpected behavior.

- You have a charts e.g. Prometheus, and each of your team wants to deploy it. And suppose it creates a clusterrole named `Prometheus`, so now the issue is when the first team deploys this chart, a ClusterRole will be created named `Prometheus` but when any other team wants to deploy the same chart it will give error, saying `A clusterrole with name Prometheus already exits` failing your release.