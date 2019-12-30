# Helm Upgrade Command Issue

There is a catch in the helm upgrade command that if the chart version is not specified, it will pull the latest version of chart and it is a default behaviour.


## Problem
```bash
helm upgrade -i --wait --force helm-operator fluxcd/helm-operator --namespace $FLUXNAMESPACE --set createCRD=true,serviceAccount.name=helm-operator,clusterRole.name=helm-operator
```

In the above command we didn't mentioned the version of the chart and it pulled the latest version 0.0.4. It cause out deployments to fail.


## Solution

```bash
helm upgrade --version 0.2.0 -i --wait --force helm-operator fluxcd/helm-operator --namespace $FLUXNAMESPACE --set createCRD=true,serviceAccount.name=helm-operator,clusterRole.name=helm-operator
```

In the command given below we have specified the version of chart to be 0.0.2.
