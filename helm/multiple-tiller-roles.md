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

## Issues with above approach

The above seems to be a better approach than giving a cluster-admin role to tiller as it restricts permissions to the tiller user, but still there is an issue as we have given it the access to create, update, delete clusterroles so it can delete a clusterrole that it hasn't created and some other team might have created that, leaving your cluster in a bad state.

But we can't get the ownership of objects and therefore can't work that way as the problem is that Ownership of objects by creator is not tracked or available to the authorization layer for decision making. as Tracking ownership is problematic in a lot of dimensions - exposure of personally identifiable information in API objects, use cases in both directions about whether creation of an object *should* confer special privileges on the object, trustworthiness of the fields storing the creator, potential for misuse of the "created by" data on an object created by a privileged user then heavily modified by an unprivileged one, etc.

What we can do is create an Admission Controller, and validate there, you could look into granting broader RBAC permissions, then restricting writes using a policy admission webhook
Something like OPA
https://www.openpolicyagent.org/docs/kubernetes-admission-control.html

You can create an Admission Contoller according to your policies in your org, e.g. i guess one namespace == one team, the authorizer then therefore could check on namespace.

## Secure Approaches

### TLS

One approach can be to create a global tiller for apps that needs escalated permissions e.g. Prometheus, Grafana which will be creating ClusterRole, ClusterRoleBinding, etc and other tillers per namespace. For global, we can secure it using tls. https://github.com/helm/helm/blob/master/docs/securing_installation.md goes through securing a tiller. Steps are:

- Creating Certificates Authority

- Create two certificate using that CA. One for tiller `tiller.cert.pem` and other for helm user `helm.cert.pem`.

- Use these certificates to initialize helm. The command will be:

``` bash
helm init \
--override 'spec.template.spec.containers[0].command'='{/tiller,--storage=secret}' \
--tiller-tls \
--tiller-tls-verify \
--tiller-tls-cert=cert.pem \
--tiller-tls-key=key.pem \
--tls-ca-cert=ca.pem \
--service-account=accountname
```

and when runnning helm command, use:

`helm ls --tls --tls-ca-cert ca.cert.pem --tls-cert helm.cert.pem --tls-key helm.key.pem`

So only the user with access to these certificates can access helm.

### RBAC

Other way can be we create a single global tiller, and then create multiple tillers per namespace. Using this way, we can have the global tiller have a `cluster-admin` role which can be used to deploy applications which create clusterroles or clusterrolebindings. And for the other tillers, we will give them permissions only for their own namespace, so they cannot create/update/delete resources of other namespaces. But the question rises that whether the user would be able to use the global helm as well as helm for his own namespace? e.g. a user A has permissions over namespace `dev` and global tiller is installed in `tiller` namespace and dev namespace also has tiller installed with limited permissions. So would user A be able to run both of these commands?
`helm ls --tiller-namespace dev`
`helm ls --tiller-namespace tiller` ?

So the answer is we will restrict the `tiller` namespace using RBAC, we will only let `cluster-admin` modify it. So in above case, the user A would not have access to `tiller` namespace so it can't run `helm ls --tiller-namespace tiller`, if user tries it, he would get an error `Error: User "A" cannot list pods in the namespace "tiller": User "A" cannot list pods in project "tiller" (get pods)`