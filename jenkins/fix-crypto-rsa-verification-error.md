# How to fix crypto/rsa: verification error

## Problem:

After creating a new cluster, when we run pipelines for umbrella charts e.g. StakaterkubeCiCd, we may face below error

```text
certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```

## Solution

Update the `k8s-current-cluster-kubeconfig` in `cicd-tools/values/secrets.yaml` in  `StakaterkubeCiCd` repo with the latest kube config. And run the pipeline.