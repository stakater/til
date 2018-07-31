## Delete all pods starting name fluentd in particular namespace

This command will delete all pods in tools namespace with name fluentd in it:

```kubectl delete pods $(kubectl get pods --namespace tools | grep fluentd | awk '{print $1}') --namespace tools```

Change the namespace name.
