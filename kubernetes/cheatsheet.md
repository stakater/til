## Export Resource

You can strip kubernetes stuff from a deployment when you export it to yaml or json

just add the --export flag, example:

```
kubectl get deploy demo -n dev -oyaml --export
```

## Delete Pod in Particular State

```
NAMESPACE=your-namespace; kubectl delete pods $(kubectl get pods --namespace $NAMESPACE | grep Evicted | awk '{ print $1 }') --namespace $NAMESPACE
```

