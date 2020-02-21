# Kuberentes how to check if I am admin?

If running all the following commands returns yes, you are most likely admin:

```
kubectl auth can-i "*" "*" --all-namespaces
kubectl auth can-i "*" namespace
kubectl auth can-i "*" clusterrole
kubectl auth can-i "*" crd
```
