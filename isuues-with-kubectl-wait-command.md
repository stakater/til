# Issue with kubectl wait command

Client Version: 1.16

Server Version: 1.14

`kubectl wait` command waits for a specific resource and checks the condition specified in the arguments and returns if the condition is met until it times out (default 30s)

## Issue
`kubectl wait` showing irregular behaviour when used to wait for HelmReleases/Pods being deployed.

- The command keeps waiting but the condition is actually satisfied. (Confirmed with `dashboard` and `kubectl get` command ) and exits with a timeout error

Command:
```
kubectl wait hr --all=true --for=condition=Released --timeout=$(TIMEOUT) --namespace $(NAMESPACE)
kubectl wait pods --all=true --for=condition=Ready --timeout=$(TIMEOUT) --namespace $(NAMESPACE)
```
