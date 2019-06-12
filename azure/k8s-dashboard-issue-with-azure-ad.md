# How Azure Kubernetes Service was deployed

AKS is deployed with Active directory by installing the Active Directory Server and Client applications first and then providing secrets/ tenants ids etc. in the command to provision a kubernetes cluster with Azure Active directory.

Following command is used to provision AKS cluster
```
az aks create \
  --resource-group <RESOURCE_GROUP> \
  --name <AKS_CLUSTER_NAME> \
  --generate-ssh-keys \
  --aad-server-app-id <SERVER_APP_ID> \
  --aad-server-app-secret <SERVER_APP_SECRET> \
  --aad-client-app-id <CLIENT_APP_ID> \
  --aad-tenant-id <TENANT_ID>
```

# How to use AKS with AD to limit access using kubectl

When AKS cluster is deployed only admin has full access to the cluster. So we need to use the `az aks get-credentials` command with the `--admin` argument to sign in to the cluster with admin access.
```
az aks get-credentials --resource-group <RESOURCE_GROUP> --name <AKS_CLUSTER_NAME> --admin
```

In order to give a user permission to list pods. we need to create a Role and then create a ClusterRoleBinding for that user.

A role `pod-reader` which can `get`, `watch` and `list` pods:
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules: 
- apiGroups: [“”] # “” indicates the core API group
  resources: [“pods”]
  verbs: [“get”, “watch”, “list”]
```

A ClusterRoleBinding which will bind a user to the role `pod-reader`:
```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: ali-view
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view
subjects:
  - kind: User
    name: <USER_ID_FROM_AZURE_ACTIVE_DIRECTORY>
    apiGroup: rbac.authorization.k8s.io
```
apply these configurations using `kubectl apply -f` command.

Now the `user` with `<USER_ID_FROM_AZURE_ACTIVE_DIRECTORY>` ID will have a restricted access to list watch and get pods. Similarly other access controls can be applied on Services, Pods and other resources etc. using `RoleBinding/ClusterRoleBinding` on `Roles` as per requirement.

# Why dashboard cannot be exposed via Ingress using Keycloak

## Stakater Workflow
Workflow to access Dashboard is as follows

```
User -->  KeyCloak-Proxy  -->  Dashboard --> Kube-api-server
            |    ^
            V    |
           KeyCloak
            |    ^
            V    |
          OAuth Server
```

kube-api-server tells the dashboard which user is accessing the dashboard and what are its access permissions by looking at the token forwarded by the keycloak during the authentication process. For this, kube-api-server needs to know about the authentication service i.e. OpenID Connect (OIDC) which is passed in the parameters:`--oidc-issuer-url` and `--oidc-client-id`. These are passed used when initializing the kubernetes api server.

## Issue in Azure Kubernetes Server (AKS)

In AKS the server/kube-api-server is managed by the Azure itself and hence we cannot pass these parameters i.e. `--oidc-issuer-url ` and `--oidc-client-id` to refer to the OIDC client. The provided API command for AKS `az aks create` command referenced [here](https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-create) does not support oidc connections as of now. Therefore we cannot pass oidc parameters to the AKS service.

kube-api-server managed by the Azure itself has no knowledge of oidc client and hence it cannot provide access contol based on the KeyCloak authentication token for dashboard service.
