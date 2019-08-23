# Flux Installation and Debugging

## Overview

[Flux](https://github.com/fluxcd/flux) is most useful when used as a deployment tool at the end of a Continuous Delivery pipeline. Flux will make sure that your new container images and config changes are propagated to the cluster.

## Installation

Flux can be installed using the following commands:

```sh

helm init --client-only

helm repo add fluxcd https://fluxcd.github.io/flux

helm repo update

kubectl apply -f rbac-flux.yaml

helm upgrade --install pliro-dev-flux --namespace $(NAMESPACE) fluxcd/flux -f flux-values.yaml

# to provide the specific chart version

helm upgrade --install pliro-dev-flux --version 0.10.1 --namespace $(NAMESPACE) fluxcd/flux -f flux-values.yaml
```

Once the above commands runs, in the end it will print a command in the console, which when runs will provide the public key that can be added inside the git proivder ssh keys. So latest application can be deployed



## Details

In the [`Chart.yaml`](https://github.com/fluxcd/flux/blob/master/chart/flux/Chart.yaml) file, appVersion is the version of the app image and the version of the chart. 

At instances the chart doesn't get updated because the local repository is not synced with the remote repository, to sync the local repository run this command:

```
helm repo update
```

## Usage in application

To enable an application for flux we need to pass the following annotations

```
flux.weave.works/automated: "true"
flux.weave.works/tag.deployment: regexp:^([0-9]+.[0-9]+.[0-9]+-develop-[0-9]+-XXX)$
```

Image manifest should be in this format

```
image:
  repository: image-repository
  tag: 0.0.0-develop-1-XXX
```
