# Monitor External services

Prometheus can monitor an external service if we create Endpoints, Service and a ServiceMonitor for it. Lets say we have an external service running on domain `test.com` at port `8080`, then we have to create the following resources in order for prometheus to monitor it:

## Endpoints

The Endpoints should look like this:

```yaml
kind: Endpoints
apiVersion: v1
metadata:
  name: test
  labels:
    app: test
    k8sapp: test
subsets:
- addresses:
  - ip: 10.0.0.1 # IP of test.com
  ports:
  - name: 8080-tcp
    port: 8080
    protocol: TCP
```

## Service

The Service will should look like this:

```yaml
kind: Service
apiVersion: v1
metadata:
  name: test
  labels:
    app: test
    k8sapp: test
spec:
  ports:
  - name: "8080-tcp"
    protocol: "TCP"
    port: 8080
    targetPort: 8080
selector: {}
```

## ServiceMonitor

Finally we can create the ServiceMonitor similar to what we do for normal Kubernetes / OpenShift services

```yaml
  - metadata:
      name: test
      labels:
        k8sapp: test
    spec:
      selector:
        matchLabels:
          app: test
      namespaceSelector:
        matchNames:
        - @{NAMESPACE-NAME}
      endpoints:
      - port: 8080-tcp
        path: /admin/prometheus # Route for prometheus
        interval: 10s
        scheme: http
        tlsConfig:
          insecureSkipVerify: true
```