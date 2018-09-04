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
    - services
  verbs: ["create","update","delete","get","list"]
- apiGroups: ["*"]
  resources:
    - endpoints
    - pods
    - nodes/metrics
    - nodes  
    - resourcequotas
    - replicationcontrollers
    - limitranges
    - persistentvolumeclaims
    - persistentvolumes
    - namespaces
    - daemonsets
    - deployments
    - replicasets
    - statefulsets
    - cronjobs
    - jobs
    - horizontalpodautoscalers
    - configmaps
  verbs: ["get", "watch", "list"]
- apiGroups: ["*"]
  resources:
    - subjectaccessreviews
    - tokenreviews
    - selfsubjectaccessreviews
  verbs: ["create", "update","delete","get", "watch", "list"]
- nonResourceURLs: ["*"]
  verbs: ["*"]
```

The above Role gives its permissions to create/update/delete any resource only in its own namespace. The 1st ClusterRole gives it permissions to create/update/delete clusterroles, clusterrolebindings,roles, rolebindings, serviceaccounts and the 2nd ClusterRole gives it permissions just to list pods, services, configmaps, etc. Tiller will not be able to create or delete pods or other resources in other namespaces.

We have explicitly defined verbs as in general, `verbs:["*"]` isn't a good permission to give to any user that isn't a superuser. Giving a [*] verb gives the user escalated permissions and the user can create any clusterrole, which can have any permissions. However giving `["create","update"]` for a (cluster)role it validates that it doesn’t contain any permissions the user doesn’t already have. Actually, the RBAC API prevents users from escalating privileges by editing roles or role bindings. Because this is enforced at the API level, it applies even when the RBAC authorizer is not in use.

A user can only create/update a role if they already have all the permissions contained in the role, at the same scope as the role (cluster-wide for a ClusterRole, within the same namespace or cluster-wide for a Role). For example, if “user-1” does not have the ability to list secrets cluster-wide, they cannot create a ClusterRole containing that permission.