# Dashboard RBAC with Keycloak

Normally when Kubernetes dashboard is configured with Keycloak, it fails to apply RBAC rules for users because the Kubernetes-dashboard-proxy uses kubernetes-api-server to determine the access rights. But kubernetes-api-server has no way of knowing which users groups are granted which access rights in Keycloak. So in order to give kubernetes-api-server access to Keycloak, we pass the following parameters in kops `cluster.yaml` file

```yaml
kubeAPIServer:
    oidcClientID: stakater-online-platform
    oidcGroupsClaim: groups
    oidcIssuerURL: https://keycloak.global.stakater.com/auth/realms/stakater
    oidcUsernameClaim: email
```

In KeyCloak Create a client with the following parameters:
* Client ID: stakater-online-platform
* Client Protocol: openid-connect
* Access Type: confidential
* Valid Redirect URIs: "*"

and create a Mapper with the following parameters:
* Name: groups
* Mapper Type: Group Membership
* Token Claim Name: groups

Now the Keycloak becomes an identity provider.

Create a group named `admin` in Groups and add users who need to have admin access.
Now create a `ClusterRoleBinding` to assign this group `admin` a built-in `cluster-admin` access using the following manifest.
```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: keycloak-admin-group
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: Group
  name: /admin
```
Now add the keycloak annotations to the kubernetes-dashboard.yaml like below and apply:
```
annotations:
    authproxy.stakater.com/enabled: "true"
    authproxy.stakater.com/upstream-url: "http://127.0.0.1:9090"
    authproxy.stakater.com/source-service-name: stakater-global-kubernetes-dashboard
    authproxy.stakater.com/redirection-url: "https://dashboard.global.stakater.com"
    authproxy.stakater.com/listen: "0.0.0.0:80"
```
Now the users in `admin` group in Keycloak can now have `cluster-admin` access in Kubernetes.