# Debug Init Containers

You can start to debug your pod by:

`kubectl get pod <pod-name>`

and it will show you 

```
NAME         READY     STATUS     RESTARTS   AGE
<pod-name>   0/1       Init:1/2   0          7s
```

To see detailed view:

`kubectl describe pod <pod-name>`

For example, a Pod with two Init Containers might show the following:
```
Init Containers:
  <init-container-1>:
    Container ID:    ...
    ...
    State:           Terminated
      Reason:        Completed
      Exit Code:     0
      Started:       ...
      Finished:      ...
    Ready:           True
    Restart Count:   0
    ...
  <init-container-2>:
    Container ID:    ...
    ...
    State:           Waiting
      Reason:        CrashLoopBackOff
    Last State:      Terminated
      Reason:        Error
      Exit Code:     1
      Started:       ...
      Finished:      ...
    Ready:           False
    Restart Count:   3
    ...
```

You can access the Init Container statuses programmatically by reading the status.initContainerStatuses field on the Pod Spec:

kubectl get pod nginx --template '{{.status.initContainerStatuses}}'
This command will return the same information as above in raw JSON.

## Accessing logs from Init Containers

Pass the Init Container name along with the Pod name to access its logs.

`kubectl logs <pod-name> -c <init-container-2>`


Source: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-init-containers/