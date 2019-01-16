## Export Resource

You can strip kubernetes stuff from a deployment when you export it to yaml or json

just add the --export flag, example:

```
kubectl get deploy demo -n dev -oyaml --export
```
