# Managing Project Networks for Openshift

In openshift, two namespaces/projects cannot communicate with each other by default, you have to create some project network for that.

When your cluster is configured to use the ovs-multitenant SDN plug-in, you can manage the separate pod overlay networks for projects using the administrator CLI.

## Joining Project Networks

To join projects to an existing project network:

```bash
oc adm pod-network join-projects --to=<project1> <project2> <project3>
```

In the above example, all the pods and services in `project2` and `project3` can now access any pods and services in `project1` and vice versa. 

## Isolating Project Networks

To isolate the project network in the cluster and vice versa, run:

```bash
oc adm pod-network isolate-projects <project1> <project2>
```

In the above example, all of the pods and services in `project1` and `project2` can not access any pods and services from other non-global projects in the cluster and vice versa.

## Making Project Networks Global

To allow projects to access all pods and services in the cluster and vice versa:

```bash
oc adm pod-network make-projects-global <project1> <project2>
```

In the above example, all the pods and services in `project1` and `project2` can now access any pods and services in the cluster and vice versa.


For above examples, instead of specifying specific project names, you can also use the --selector=`project_selector` option.

### Reference

https://docs.openshift.com/container-platform/3.5/admin_guide/managing_networking.html#admin-guide-pod-network