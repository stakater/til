# How Prometheus Operator & Prometheus work

When deploying Prometheus through Prometheus Operator, Prometheus Operator creates a CRD of Prometheus and we deploy the Prometheus object, PO(Prometheus Operator) makes a Stateful Set of this Prometheus object which creates Pods. The naming and labels of this pod is handled by Prometheus Operator.

We also create a Service for Prometheus to expose Prometheus pods. To do so, we use label selectors that matches the labels on the Pod to access Prometheus and read data from it. To understand how this labelling and naming convention work see below sample Prometheus object and its corresponding sSrvice.

## Prometheus.yaml

``` yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: k8s
  labels:
    prometheus: ocp-supporting-services
    app: prometheus
    group: com.stakater.platform
    provider: stakater
    version: "2.2.0-rc.0"
    chart: "prometheus-1.0.15"
    release: "RELEASE-NAME"
    heritage: "Tiller"
spec:
  replicas: 2
  version: v2.2.0-rc.0
  externalUrl: http://127.0.0.1:9090
  serviceAccountName: monitoring-k8s
  serviceMonitorSelector:
    matchExpressions:
    - {key: k8s-app, operator: Exists}
  ruleSelector:
    matchLabels:
      app: prometheus
      group: com.stakater.platform
      provider: stakater  
  alerting:
    alertmanagers:
    - namespace: monitoring
      name: alertmanager-main
      port: web
```

## Service.yaml

``` yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    prometheus: ocp-supporting-services    
    app: prometheus
    group: com.stakater.platform
    provider: stakater
    version: "2.2.0-rc.0"
    chart: "prometheus-1.0.15"
    release: "RELEASE-NAME"
    heritage: "Tiller"
  name: prometheus-k8s  
spec:
  ports:
  - name: web
    port: 9090
    protocol: TCP
    targetPort: web
  selector:
    prometheus: k8s
    app: prometheus
```

The details of the fields in `Prometheus` and the labelling convention are given below:

### Name

The field `name` in Prometheus is used as the Pod name appended with `prometheus-<name>-<replica-no>` e.g. in the above case the pod name will be `prometheus-k8s-0`. This name is also used as label on the Pod made by PO, i.e. the Pod `prometheus-k8s-0` will have labels

``` yaml
 prometheus: k8s    # From Prometheus name
 app: prometheus    # Not configurable
```

We can configure the `prometheus: <name>` label by changing the name but cannot change `app: prometheus` label. So in the Service, you can see we have added 

```yaml
prometheus: k8s
app: prometheus
```

as selectors

### Labels

 The labels provided in Prometheus object are added on the Stateful Set made by PO but not on the Pods.

### ServiceMonitorSelector

 This will be used to select Service Monitors for this Prometheus e.g. in this case , the service monitors should have `k8s-app` key as label.

### RuleSelector

In the Prometheus object described above, there is a ruleSelector field, which selects the ConfigMaps from which .rules files are loaded. This means that for a ConfigMap to be loaded by the Prometheus object it should have these labels.

### Alerting

This will be used for the alertmanager, e.g. in above Prometheus an  alertmanager with name `alertmanager-main` in namespace `monitoring` is used.