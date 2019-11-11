# Rancher Project Limitation

Rancher introduced new object `Project`, projects provide us the capability to manage multiple namespaces as a single entity. 
The new heirarcy being : 
* Cluster contains projects
* Projects contains(are associated with) namespaces

## To associate a namespace with a project there are 2 possible ways: 

1. Using rancher CLI: 
By runing the following command `rancher namespaces move <namespace-name> <clusterId>:<projectId>` we can associate namespace
to a particular project

2. Annotating the namespace(Kubectl): 
The annotation `field.cattle.io/projectId: <clusterId:projectId>` can be used to associate a namespace with a project.
Corresponding manifest will look something like this: 

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: namespace-name
  namespace: namespace-name
  labels:
    name: namespace-name
  annotations:
    field.cattle.io/projectId: <clusterId>:<projectId>
```

Or else you can run the following command 
`kubectl annotate namespace namespace-name field.cattle.io/projectId=<clusterId>:<projectId>` 

## *The issue*: 
Both of these seem like viable solution but both add a redundant dependecy of installing `Rancher CLI`. Since, projects are 
rancher managed objects and don't exist in kubernetes own realm *currently* there is no way of retrieving the projectId 
without using rancher cli. To retrieve project id using rancher cli use the following script:

```
NAMESPACES_TO_ANNOTATE="dev tools monitoring"
rancher login <rancher-url> --token <access-token>
PROJECT_ID=$(rancher projects | grep '<project-name' | awk '{print $1}') #Use 'Default' if you don't know the project name
# Annotate Namespaces to associate them with a project
for NAMESPACE in NAMESPACES_TO_ANNOTATE; do kubectl annotate namespace $NAMESPACE field.cattle.io/projectId=$PROJECT_ID; done
```

*NOTE:* Kindly, update this document as soon as there is any rancher cli independant way of associating namespaces with 
projects.

Relevant links:
https://rancher.com/docs/rancher/v2.x/en/cluster-admin/projects-and-namespaces/
https://github.com/rancher/rancher/issues/16493
https://github.com/rancher/rancher/issues/23376
