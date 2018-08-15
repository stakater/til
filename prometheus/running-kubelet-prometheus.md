# Accessing Kubelet through Prometheus

For prometheus to scrape data from Kubelet, you need to add Service Monitor for Prometheus.

## Service Monitors for Kubelet

```yaml
- metadata:
    name: kubelet
    labels:
    k8sapp: kubelet
spec:
    jobLabel: k8s-app
    selector:
    matchLabels:
        k8s-app: kubelet
    namespaceSelector:
    matchNames:
    - kube-system
    endpoints:
    - port: cadvisor
    interval: 30s
    honorLabels: true
```

Add the above service monitor for Kubelet and access Prometheus Dashboard, go to Status->Targets and see if the kubelet target is up or not. If it is not, then it might be due to some endpoint problem. As in 1.7.3+ versions, the port and path has been changed. So change the endpoint in above service monitor

```yaml
    endpoints:
    - port: cadvisor
    interval: 30s
    honorLabels: true
```

to

```yaml
    endpoints:
    - port: http-metrics
    path: metrics/cadvisor
    interval: 30s
    honorLabels: true
```

Check prometheus dashboard again if the kubelets are up or not, If still they are not up and it gives error related to `refused connection socket`, then change the endpoint to

```yaml
    endpoints:
    - port: https-metrics
    scheme: https
    tlsConfig:
        caFile: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    path: metrics/cadvisor
    interval: 30s
    honorLabels: true
```

And check the Prometheus dashboard, it should be able to access Kubelets via one of the above endpoints.