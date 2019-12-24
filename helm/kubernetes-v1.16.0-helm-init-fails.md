# Helm init fails against kubernetes v1.16.0

The error:

```
$ helm init --service-account tiller
$HELM_HOME has been configured at /Users/Waleed/.helm.
Error: error installing: the server could not find the requested resource
```

Current workaround is to update tiller's deployment and change apiVersion from `extensions/v1beta1` to `apps/v1`. You can 
initialize helm as:

`helm init --service-account tiller --output yaml | sed 's@apiVersion: extensions/v1beta1@apiVersion: apps/v1@' | sed 's@  replicas: 1@  replicas: 1\n  selector: {"matchLabels": {"app": "helm", "name": "tiller"}}@' | kubectl apply -f -
`
