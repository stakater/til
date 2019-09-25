# Helm

## Deleting all releases in a specific k8s namespace
```
helm ls --all --short --namespace $(NAMESPACE) | xargs -L1 helm delete --purge
```