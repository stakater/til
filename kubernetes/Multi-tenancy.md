# Multi-tenancy

## What is multi-tenancy?

A multi-tenant cluster is shared by multiple users and/or workloads which are referred to as "tenants". The operators of multi-tenant clusters must isolate tenants from each other to minimize the damage that a compromised or malicious tenant can do to the cluster and other tenants. Also, cluster resources must be fairly allocated among tenants.

When you plan a multi-tenant architecture you should consider the layers of resource isolation in Kubernetes: 

- cluster, 
- namespace, 
- node, 
- pod, and 
- container. 

You should also consider the security implications of sharing different types of resources among tenants. For example, scheduling pods from different tenants on the same node could reduce the number of machines needed in the cluster. On the other hand, you might need to prevent certain workloads from being colocated. For example, you might not allow untrusted code from outside of your organization to run on the same node as containers that process sensitive information.

Although Kubernetes cannot guarantee perfectly secure isolation between tenants, it does offer features that may be sufficient for specific use cases. You can separate each tenant and their Kubernetes resources into their own namespaces. You can then use policies to enforce tenant isolation. Policies are usually scoped by namespace and can be used to restrict API access, to constrain resource usage, and to restrict what containers are allowed to do.

The tenants of a multi-tenant cluster share:

- Extensions, controllers, add-ons, and custom resource definitions (CRDs).
- The cluster control plane. This implies that the cluster operations, security, and auditing are centralized.

Operating a multi-tenant cluster has several advantages over operating multiple, single-tenant clusters:

- Reduced management overhead
- Reduced resource fragmentation
- No need to wait for cluster creation for new tenants

## Multi-tenancy use cases

This section describes how you could configure a cluster for various multi-tenancy use cases.

### Enterprise multi-tenancy

In an enterprise environment, the tenants of a cluster are distinct teams within the organization. Typically, each tenant has a corresponding namespace. Network traffic within a namespace is unrestricted. Network traffic between namespaces must be explicitly whitelisted. These policies can be enforced using Kubernetes network policy.

The users of the cluster are divided into three different roles, depending on their privilege:

- Cluster administrator: This role is for administrators of the entire cluster, who manage all tenants. Cluster administrators can create, read, update, and delete any policy object. They can create namespaces and assign them to namespace administrators.
- Namespace administrator: This role is for administrators of specific, single tenants. A namespace administrator can manage the users in their namespace.
- Developer: Members of this role can create, read, update, and delete namespaced non-policy objects like Pods, Jobs, and Ingresses. Developers only have these privileges in the namespaces they have access to.

## Multi-tenancy policy enforcement

Kubernetes provide several features that can be used to manage multi-tenant clusters. The following sections give an overview of these features.

### 1. Access control

RBAC is built into Kubernetes and grants granular permissions for specific resources and operations within your clusters.

### 2. Network policies

Cluster network policies give you control over the communication between your cluster's Pods. Policies specify which namespaces, labels, and IP address ranges a Pod can communicate with.

### 3. Resource quotas

Resource quotas manage the amount of resources used by the objects in a namespace. You can set quotas in terms of CPU and memory usage, or in terms of object counts. Resource quotas let you ensure that no tenant uses more than its assigned share of cluster resources.

### 4. Pod security policies

PodSecurityPolicies are a Kubernetes API type that validate requests to create and update Pods. PodSecurityPolicies define default values and requirements for security-sensitive fields of Pod specification. You can create policies that restrict the deployment of Pods that access the host filesystem, networks, PID namespaces, volumes, and more.

A PodSecurityPolicy is an admission controller resource you create that validates requests to create and update Pods on your cluster. The PodSecurityPolicy defines a set of conditions that Pods must meet to be accepted by the cluster; when a request to create or update a Pod does not meet the conditions in the PodSecurityPolicy, that request is rejected and an error is returned.

To use PodSecurityPolicy, you must first create and define policies that new and updated Pods must meet. Then, you must enable the PodSecurityPolicy admission controller, which validates requests to create and update Pods against the defined policies.

When multiple PodSecurityPolicies are available, the admission controller uses the first policy that successfully validates. Policies are ordered alphabetically, and the controller prefers non-mutating policies (policies that don't change the Pod) over mutating policies.

