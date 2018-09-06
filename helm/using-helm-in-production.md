# Using Helm in Production

Helm is a Production Ready Chart manager which can be used to define, install, and upgrade even the most complex Kubernetes application. Helm has two main components

- Tiller: Server side component, it creates a deployment
- Helm: Client Side component, it is used to communicate to the server and deploy our apps

For tiller, you create a deployment and correspond a service account to that deployment, and you can give permissions to the service account according to your needs and deploy things through this.

## Approaches

### Single Tiller

We can setup a single tiller for the whole cluster and give that tiller `cluster-admin` role and each of the apps can be deployed through that single tiller. Usually, tiller is installed in `kube-system` namespace. This is not a suitable case if you have multiple teams working on a single cluster because each team would be able to list, modify or even delete releases of other teams. This can only be deployed if you have a single team per cluster. You would have to create following ServiceAccount & ClusterRoleBinding:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

This can be deployed through following commands:

```bash
kubectl apply -f tiller.yaml            # above file
helm init --service-account tiller      # by default namespace is kube-system, you can specify it by passing a flag --tiller-namespace
```

It will create a deployment `tiller-deploy` in kube-system namespace. You can then install any chart through helm and check its releases by `helm ls`

### Multiple Tillers

We can have multiple tillers per namespace so that each tiller can have only access to its own namespace only. This method is best where multiple teams share a single cluster and you want that a team cannot modify or change resources of other teams. For this approach, you can create following roles and rolebindings:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller-dev
  namespace: dev
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: tiller-dev-binding
  namespace: dev
subjects:
- kind: ServiceAccount
  name: tiller-dev
  namespace: dev
roleRef:
  kind: Role
  name: tiller-dev-role
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: tiller-dev-role
  namespace: dev
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]

```

The above role grants `tiller-dev` service account complete access to `dev` namespace and it can create/update/delete resources of only `dev` namespace and cannot modify resources of other namespaces. To initialize helm, run following commands:

```bash
kubectl apply -f tiller-dev.yaml                        # above file
helm init --service-account tiller-dev --tiller-namespace dev
```

It will create a deployment `tiller-deploy` in dev namespace which will only be able to modify its own resources.

In this scenario, there is one missing use case, What if you want to deploy apps like Prometheus that create ClusterRoles or ClusterRoleBindings. You can't do that with this tiller as it is restricted to only `dev` namespace. And if you grant `tiller-dev` service account extra permisssions to modify clusterroles(bindings), so it can change clusterroles which other teams might have created.

So to overcome this sort of problem, we can have a single global tiller and per namespace tillers. The global tiller will have `cluster-admin` role and can create ClusterRoles or ClusterRoleBindings, so we can use the global tiller to deploy different stacks like Monitoring, Logging, etc or apps that need ClusterRole level permissions, wheras the teams can deploy their specific apps using the namespace specific tiller.

But the question rises that whether the dev user would be able to use the global helm as well as helm for his own namespace? Would a dev user A be able to run both of these commands?

`helm ls --tiller-namespace dev`

`helm ls --tiller-namespace tiller` ?

So the answer is we will restrict the `tiller` namespace using RBAC, we will only let `cluster-admins` modify it. So in above case, the user A would not have access to `tiller` namespace, so it can't run `helm ls --tiller-namespace tiller`, 

if user tries it, he would get an error `Error: User "A" cannot list pods in the namespace "tiller": User "A" cannot list pods in project "tiller" (get pods)`

And he can check his releases through `helm ls --tiller-namespace dev` only.