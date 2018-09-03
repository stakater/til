# Handle Multiple tillers & their Roles

We are creating multiple tillers, each for a specific namespace so that a team should not be able to mess up releases or resources of other teams but the issue is that some teams need to deploy apps like Prometheus which creates clusterroles and clusterrolebindings, so to handle this type of workflow, following Roles & ClusterRoles are used.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller-test
  namespace: tiller-test
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: tiller-binding
  namespace: tiller-test
subjects:
- kind: ServiceAccount
  name: tiller-test
  namespace: tiller-test
roleRef:
  kind: Role
  name: tiller-role
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: tiller-role
  namespace: tiller-test
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: tiller-binding
subjects:
- kind: ServiceAccount
  name: tiller-test
  namespace: tiller-test
roleRef:
  kind: ClusterRole
  name: cluster-role
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: cluster-role
rules:
- apiGroups: ["*"]
  resources:
    - clusterroles
    - clusterrolebindings
    - roles
    - rolebindings
    - serviceaccounts
  verbs: ["*"]
- apiGroups: ["*"]
  resources:
    - endpoints
    - pods
    - nodes/metrics
    - nodes
    - selfsubjectaccessreviews
    - configmaps
    - services
  verbs: ["get", "watch", "list"]
- nonResourceURLs: ["*"]
  verbs: ["*"]

```

The above Role gives its permissions to create/update/delete any resource only its own namespace. The 1st ClusterRole gives it permissions to create/update/delete clusterroles, clusterrolebindings,roles, rolebindings, serviceaccounts and the 2nd ClusterRole gives it permissions just to list pods, services, configmaps, etc. Tiller will not be able to create or delete pods or other resources in other namespaces