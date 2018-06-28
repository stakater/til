# Running Multiple Prometheus with single alertmanager

Inspired from https://coreos.com/tectonic/docs/latest/tectonic-prometheus-operator/user-guides/application-monitoring.html

Prometheus uses ServiceMonitors to check for services and scrape data out of them. It uses alertmanagers to send alerts. We can have multiple prometheus sending data to a single alertmanager.

```
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus-k8s
  namespace: monitoring
  labels:
    prometheus: monitoring-k8s
spec:
  version: v2.0.0
  externalUrl: http://127.0.0.1:9090
  serviceAccountName: monitoring-k8s
  serviceMonitorSelector:
    matchExpressions:
    - {key: k8s-app, operator: Exists}
  ruleSelector:
    matchLabels:
      prometheus: monitoring-k8s
  resources:
    requests:
      memory: 400Mi
  alerting:
    alertmanagers:
    - namespace: monitoring
      name: alertmanager-main
      port: web
```

In the above prometheus configuration, the `alerting` section actually manages the alertmanager. In above example, the alertmanager is also deployed in `monitoring` namespace, and we are deploying prometheus in `monitoring` namespace too so it will work fine. 

But if we want multiple prometheuses in other namespaces to use the same alertmanager, we will be using the same `alerting` section in all Prometheus as well.

```
  alerting:
    alertmanagers:
    - namespace: monitoring
      name: alertmanager-main
      port: web
```

In order for the Prometheus instances to be able to discover the Alertmanager cluster of the monitoring namespace, it requires the respective RBAC permissions. e.g. If we want to deploy Prometheus in **dev** namespace
```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: prometheus-dev-rolebinding
  namespace: monitoring
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: prometheus-dev-role
subjects:
- kind: ServiceAccount
  name: *service-account-of-prometheus*
  namespace: dev
```
```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: prometheus-dev-role
  namespace: monitoring
rules:
- apiGroups: [""]
  resources:
  - nodes
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources:
  - configmaps
  verbs: ["get"]
```
