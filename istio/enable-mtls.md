# Enable MTLS between services

For securing communication between microservices istio allows us to enable mutual authentication with Transport Layer 
Security(mTLS). Istio requires us to use a **"Policy"** object to instruct a service, namespace, or mesh to recieve mTLS
traffic. We then use **"DestinationRule** to define policy that ensures that all traffic intended for the service(s) uses
mTLS i.e. client services make mTLS connection with target services using appropriate certificates.

## To enable mTLS first create mTLS authentication policy: 

1. For a specific service i.e. service-name

```yaml
apiVersion: authentication.istio.io/v1alpha1
kind: Policy
metadata:
  name: default
  namespace: namespace-name
spec:
  targets:
  - name: service-name
  peers:
  - mtls: {}
```

2. To apply it name-space wide

```yaml
apiVersion: authentication.istio.io/v1alpha1
kind: Policy
metadata:
  name: default
  namespace: namespace-name
spec:
  peers:
  - mtls: {}
```

3. To apply it mesh wide:

```yaml
apiVersion: authentication.istio.io/v1alpha1
kind: MeshPolicy
metadata:
  name: default
spec:
  peers:
  - mtls: {}
```

## Create destination rule:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: default
  namespace: namespace-name
spec:
  host: "service-name.namespace-name.svc.cluster.local"
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
```

P.s. use `*.namespace-name.svc.cluster.local` for forcing mTLS on all services in that particular namespace.


## Relevant links:

https://istio.io/docs/tasks/security/authentication/mtls-migration/

https://istio.io/docs/tasks/security/authentication/mutual-tls/

https://redhat-developer-demos.github.io/istio-tutorial/istio-tutorial/1.3.x/8mTLS.html
