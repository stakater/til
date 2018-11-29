# Make Prometheus Operator read Prometheus.yml config instead of ServiceMonitor

Note that for this to work, Your PO should be greater than 0.19 version.

You have to add a Secret with a name and a file which contains your Prometheus configuration like 

## Configuration:

```yaml
- job_name: "prometheus"
  static_configs:
  - targets: ["localhost:9090"]
```

## Secret:

```yaml
apiVersion: v1
data:
  prometheus-additional.yaml: LSBqb2JfbmFtZTogInByb21ldGhldXMiCiAgc3RhdGljX2NvbmZpZ3M6CiAgLSB0YXJnZXRzOiBbImxvY2FsaG9zdDo5MDkwIl0K
kind: Secret
metadata:
  name: additional-scrape-configs
  namespace: ocp-supporting-services
type: Opaque
```

and in your prometheus.yaml file add,

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: k8s
  namespace: test
spec:
  additionalScrapeConfigs:
    key: prometheus-additional.yaml
    name: additional-scrape-configs
  alerting:
    alertmanagers:
    - name: alertmanager-main
      namespace: test
      port: web
  replicas: 1
  retention: 168h
  serviceAccountName: k8s
  serviceMonitorSelector:
    matchExpressions:
    - key: k8s-app
      operator: Exists
  storage:
    class: ssd
    volumeClaimTemplate:
      spec:
        resources:
          requests:
            storage: 40Gi
        storageClassName: ssd
  version: v2.2.0-rc.0
```

Check for spec.additionalScrapeConfigs, it will add the configurations to your Prometheus configuration. 