## Network Zones and Fine-Grained Firewall Rules

In traditional data centers, we usually find coarse-grained network security zones. Each network security zone is isolated from the others with carefully scrutinized and manually configured firewall rules. Traffic within a single zone normally flows freely.

A common set up is a three-tier network with a Demilitarized Zone (DMZ), an internal network, and a data network (in increasing degree of protection). It is not uncommon for large enterprises to have more than three network zones.

Because manual intervention in the firewall rule configuration is necessary, each organization has to strike a balance between security, which calls for smaller, more numerous network security zones, and the cost of maintaining those zones.

Software Defined Networks (SDNs) change this landscape. In fact, with SDN, firewall rule configurations can be easily automated, decreasing the cost of network operations. With SDNs, it is now conceivable to define per-host firewall rules. This concept is also known as microsegmentation.

NetworkPolicies represent firewall rules in the Kubernetes/OpenShift SDN and can be used to achieve microsegmentation.

## NetworkPolicy SDN

OpenShift installation requires you to choose the SDN implementation that is best for you. In OpenShift, available options include Subnet, Multitenant, and NetworkPolicy.

The network policy SDN features are a superset of the subnet and multitenant SDN features, making it an obvious choice for newly-built OpenShift clusters.

By default, NetworkPolicy SDN behaves like Subnet SDN, since there is no NetworkPolicy configured. If you want to recreate the multitenant SDN behavior, you need to add the following NetworkPolicy to each project:

```
kind: NetworkPolicy
apiVersion: extensions/v1beta1
metadata:
  name: allow-same-and-default-namespace
spec:
  ingress:
  - from:
    - podSelector: {}
  - from:
    - namespaceSelector:
        matchLabels:
          name: default
```

This NetworkPolicy says that pods in this namespace can accept connections from other pods in the same namespace and from the default namespace (to allow for the router connections).

You also need to label the default project as follows:

```
oc label namespace default name=default
```

The creation of this NetworkPolicy object can be automated by editing the OpenShift project template. You can do this during installation by adding the following configuration to your Ansible inventory file:

```
openshift_project_request_template_edits:
  - key: objects
    action: append
    value:
      kind: NetworkPolicy
      apiVersion: extensions/v1beta1
      metadata:
        name: allow-same-and-default-namespace
        ...
```

## NetworkPolicy Implementation

In OpenShift, NetworkPolicies are implemented as OVS flow rules. NetworkPolicies flow rules are created in table 80. With this command (which you are required to run on one of the SDN nodes) you can view the flow rules corresponding to the defined NetworkPolicies:

```
ovs-ofctl dump-flows -O OpenFlow13 br0 | grep 80
```

For example, if you define a NetworkPolicy as follows:

```
kind: NetworkPolicy
apiVersion: extensions/v1beta1
metadata:
  name: prova
spec:
  podSelector:
    matchLabels:
      app: product-catalog
  ingress:
  - ports:
    - protocol: TCP
      port: 8080
```      
      
and have a pod that is selected by this NetworkPolicy, the output will be something like this:

```
cookie=0x0, duration=249.002s, table=80, n_packets=0, n_bytes=0, priority=150,tcp,reg1=0x6d285,nw_dst=10.128.0.67,tp_dst=8080 actions=output:NXM_NX_REG2[]
cookie=0x0, duration=248.994s, table=80, n_packets=0, n_bytes=0, priority=100,ip,reg1=0x6d285,nw_dst=10.128.0.67 actions=drop
```

where 10.128.0.67 was the address of the pod.
