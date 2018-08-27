# Monitoring Cross Namespace Applications

Prometheus can monitor applications in any namespace, and show its corresponding metrics, but for that, you need to have proper RBAC for prometheus to get endpoints of another namespace. So when monitoring applications in any other namespace, create and RBAC,

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: prometheus-rolebinding
  namespace: monitoring         # Namespace to which monitor apps in
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: prometheus-role
subjects:
- kind: ServiceAccount
  name: monitoring-k8s          # The service account of your promethues which is used for prometheus
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:  
  name: prometheus-role
  namespace: monitoring         # Namespace to which monitor apps in
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

After this, check the endpoint your service exposes for Prometheus and add that endpoint in your Service Monitor.