# Optimizing nginx

I used the following set of configuration and it helped me optimized the response time and TTFB(Time to first byte) for calls passing through nginx.

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: namespace-name
data:
  ssl-protocols: TLSv1.2
  proxy-connect-timeout: "300"
  use-http2: "false"
  worker-processes: "24"
  worker-connections: "100000"
  worker-rlimit-nofile: "102400"
  worker-cpu-affinity: "auto"
  keepalive: "200"
  hsts-preload: "true"
  hsts-max-age: "31536000"
  proxy-buffer-size: "256k"
```

Using `ssl-protocols: TLSv1.3` can improve TTFB further. 

To use this configmap:

```yaml
apiVersion: helm.fluxcd.io/v1
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
    controller:
    ...
    ...
      extraArgs:
        configmap: namespace-name/nginx-configuration
``` 
