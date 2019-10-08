# Exposing TCP and UDP services using nginx-ingress

Nginx-ingress provides us with the capibility to manage http/https traffic for multiple services. But ingress doesn't support
TCP or UDP services. Although, there is a workaround, using nginx-ingress we can use the args `--tcp-services-configmap` and
`--udp-services-configmap` to map an exposed(external world) port to a corresponding kubernetes service using a configmap.
Following is an example of exposing tcp port for mongodb:

```yaml
...
spec:
   - args:
     - /nginx-ingress-controller
     - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
...
```

```yaml
...
apiVersion: v1
kind: ConfigMap
metadata:
  name: tcp-services
  namespace: namespace-name
data:
  27017: "namespace-name/mongodb-svc:27017"
...
```

Using nginx-ingress from helm charts makes it even easier:

```yaml
apiVersion: flux.weave.works/v1beta1
kind: HelmRelease
metadata:
  name: nginx-ingress
  namespace: namespace-name
spec:
  releaseName: nginx-ingress
  chart:
    repository: https://kubernetes-charts.storage.googleapis.com 
    name: nginx-ingress
    version: 1.6.10
  values:
...
...
    tcp:
      27017: "namespace-name/mongodb-svc:27017"

```
