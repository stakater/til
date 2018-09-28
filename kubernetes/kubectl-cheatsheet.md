## Delete all pods starting name fluentd in particular namespace

This command will delete all pods in tools namespace with name fluentd in it:

```
kubectl delete pods $(kubectl get pods --namespace tools | grep fluentd | awk '{print $1}') --namespace tools
```

Change the namespace name.

## Delete all ingresses in a particular namespace

This command will delete all pods in tools namespace with name fluentd in it:

```
kubectl delete ing $(kubectl get ing --namespace cp | awk '{print $1}') --namespace cp
```

Change the namespace name.

## Generate ConfigMap file from a directory

```
kubectl create configmap my-config --from-file=configuration/ -o yaml --dry-run
```
