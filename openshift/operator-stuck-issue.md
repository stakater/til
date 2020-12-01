# Operator stuck Issue Not shown in Catalog

Sometimes an operator is deployed and after applying `Subscription`, the operator is not installed and stuck in Unknown state.

# Solution

The operator is stuck because the Operator cannot be found in the catalog. To refresh the catalog Run the following command and check again:

`oc delete deployment redhat-operators certified-operators community-operators redhat-marketplace -n openshift-marketplace`

When all the pods are recreated, the missing operator will appear in the catalog and the operator would be successfully installed.


If you do not have access to the cluster do this:

- Get the Kubeconfig from the parent cluster
- Give cluster admin access to the user
```
cat <<EOF | kubectl apply -f -
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: temp-cluster-admin
subjects:
  - kind: User
    apiGroup: rbac.authorization.k8s.io
    name: usama@stakater.com
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
EOF
```
- Debug the issue