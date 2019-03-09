# OpenShift Architecture

## Architectural Overview

Red Hat OpenShift Container Platform is managed by the Kubernetes container orchestrator, which manages containerized applications across a cluster of systems running the Docker container runtime. The physical configuration of Red Hat OpenShift Container Platform is based on the Kubernetes cluster architecture. OpenShift is a layered system designed to expose underlying Docker-formatted container image and Kubernetes concepts as accurately as possible, with a focus on easy composition of applications by a developer. For example, install Ruby, push code, and add MySQL. The concept of an application as a separate object is removed in favor of more flexible composition of "services", allowing two web containers to reuse a database or expose a database directly to the edge of the network.

This Red Hat OpenShift Container Platform reference architecture contains five types of nodes: 

- bastion, 
- master, 
- infrastructure, 
- storage, and 
- application.

## Bastion Node

This is a dedicated node that serves as the main deployment and management server for the Red Hat OpenShift cluster. It is used as the logon node for the cluster administrators to perform the system deployment and management operations, such as running the Ansible OpenShift deployment Playbooks and performing scale-out operations. Also, Bastion node runs DNS services for the OpenShift Cluster nodes. The bastion node runs Red Hat Enterprise Linux 7.5 can either be on bare metal or a VM running on existing vSphere environment

## Kubernetes Infrastructure

Within OpenShift Container Platform, Kubernetes manages containerized applications across a set of Docker runtime hosts and provides mechanisms for deployment, maintenance, and application-scaling. The Docker service packages, instantiates, and runs containerized applications.

A Kubernetes cluster consists of one or more masters and a set of nodes. This solution design includes HA functionality at the hardware as well as the software stack. Kubernetes cluster is designed to run in HA mode with 3 master nodes and 2 Infra nodes to ensure that the cluster has no single point of failure.

## OpenShift Master Nodes

The OpenShift Container Platform master is a server that performs control functions for the whole cluster environment. It is responsible for the creation, scheduling, and management of all objects specific to Red Hat OpenShift. It includes API, controller manager, and scheduler capabilities in one OpenShift binary. It is also a common practice to install an etcd key-value store on OpenShift masters to achieve a low-latency link between etcd and OpenShift masters. It is recommended that you run both Red Hat OpenShift masters and etcd in highly available environments. This can be achieved by running multiple OpenShift masters in conjunction with an external active-passive load balancer and the clustering functions of etcd. The master manages nodes in the Kubernetes cluster and schedules the pods (single/group of containers) to run on the application nodes. The OpenShift master node runs Red Hat Enterprise Linux 7.5.

### Master components

- API Server: The Kubernetes API server validates and configures the data for pods, services, and replication controllers. It also assigns pods to nodes and synchronizes pod information with service configuration. Can be run as a standalone process.
- etcd: etcd stores the persistent master state while other components watch etcd for changes to bring themselves into the desired state. etcd can be optionally configured for high availability, typically deployed with 2n+1 peer services.
- Controller Manager Server: The controller manager server watches etcd for changes to replication controller objects and then uses the API to enforce the desired state. Can be run as a standalone process. Several such processes create a cluster with one active leader at a time.

To mitigate concerns about availability of the master, the solution design uses a high-availability solution to configure masters and ensure that the cluster has no single point of failure. When using the native HA method with HAProxy, master components have the following availability:

###  Availability Matrix with HAProxy

| Role | Style | Description |
| --- | --- | --- |
| etcd | Active-Active | Fully redundant deployment with load balancing |
| API Server | Active-Active | Managed by HAProxy |
| Controller Manager Server | Active-Passive | One instance is elected as a cluster leader at a time |
| HAProxy | Active-Passive | Balances load between API master endpoints |




