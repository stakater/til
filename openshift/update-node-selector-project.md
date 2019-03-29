# Updating Node selector for a project

In openshift, if you create a project and deploy a pod, it will deploy that on compute nodes by default.

If you want to deploy your pod on infra nodes or master, you would have to update node selector of the project you are deploying that pod in.

`oc patch namespace <proj-name> -p '{"metadata":{"annotations":{"openshift.io/node-selector":"node-role.kubernetes.io/infra=true"}}}'`

From this, the pod will be created on infra nodes by default.
