# Helm Release Deployed but not Installed

I deleted the existing platform compnents using the Gitlab CI pipeline. I faced an issue in which helm releases were deployed by flux but when all the things were deployed. I checked the helm release state and all of them were in `DEPLOYED` state but the `message` column was empty, I used the command given below to the the state:

```bash
sudo kubectl get hr --all-namespaces
```

When we looked at the logs of:

1. Helm Operator: It was showing that the helm releases have been processed successfully.

2. Flux: It was apply the helm release.


It means the the above two tools were working fine. But the pods were not being created.

### Hypothesis

1. One reason can be the sync interval of flux, which was 10 seconds which was causing the helm releases to be processed again and again. 



### Current Fix

Its current fix is that when I delete the release in all namespaces, then they get deployed and installed successfully.

```bash
sudo kubectl delete hr --all -n <namespace>
```





