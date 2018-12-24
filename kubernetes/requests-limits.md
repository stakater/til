# Resource requests and limits

When Kubernetes schedules a Pod, it’s important that the containers have enough resources to actually run. If you schedule a large application on a node with limited resources, it is possible for the node to run out of memory or CPU resources and for things to stop working!

It’s also possible for applications to take up more resources than they should. This could be caused by a team spinning up more replicas than they need to artificially decrease latency (hey, it’s easier to spin up more copies than make your code more efficient!), to a bad configuration change that causes a program to go out of control and use 100% of the available CPU. Regardless of whether the issue is caused by a bad developer, bad code, or bad luck, what’s important is that you be in control.

## Requests and Limits

Requests and limits are the mechanisms Kubernetes uses to control resources such as CPU and memory. Requests are what the container is guaranteed to get. If a container requests a resource, Kubernetes will only schedule it on a node that can give it that resource. Limits, on the other hand, make sure a container never goes above a certain value. The container is only allowed to go up to the limit, and then it is restricted.

It is important to remember that the limit can never be lower than the request. If you try this, Kubernetes will throw an error and won’t let you run the container.

Requests and limits are on a per-container basis. While Pods usually contain a single container, it’s common to see Pods with multiple containers as well. Each container in the Pod gets its own individual limit and request, but because Pods are always scheduled as a group, you need to add the limits and requests for each container together to get an aggregate value for the Pod.

To control what requests and limits a container can have, you can set quotas at the Container level and at the Namespace level.

## Container settings

There are two types of resources: CPU and Memory. The Kubernetes scheduler uses these to figure out where to run your pods.

Each container in the Pod can set its own requests and limits, and these are all additive. So in the above example, the Pod has a total request of 500 mCPU and 128 MiB of memory, and a total limit of 1 CPU and 256MiB of memory.

### CPU

CPU resources are defined in millicores. If your container needs two full cores to run, you would put the value “2000m”. If your container only needs ¼ of a core, you would put a value of “250m”.

One thing to keep in mind about CPU requests is that if you put in a value larger than the core count of your biggest node, your pod will never be scheduled. Let’s say you have a pod that needs four cores, but your Kubernetes cluster is comprised of dual core VMs—your pod will never be scheduled!

Unless your app is specifically designed to take advantage of multiple cores (scientific computing and some databases come to mind), it is usually a best practice to keep the CPU request at ‘1’ or below, and run more replicas to scale it out. This gives the system more flexibility and reliability.

It’s when it comes to CPU limits that things get interesting. CPU is considered a “compressible” resource. If your app starts hitting your CPU limits, Kubernetes starts throttling your container. This means the CPU will be artificially restricted, giving your app potentially worse performance! However, it won’t be terminated or evicted. You can use a liveness health check to make sure performance has not been impacted.

### Memory

Memory resources are defined in bytes. Normally, you give a mebibyte value for memory (this is basically the same thing as a megabyte), but you can give anything from bytes to petabytes.

Just like CPU, if you put in a memory request that is larger than the amount of memory on your nodes, the pod will never be scheduled.

Unlike CPU resources, memory cannot be compressed. Because there is no way to throttle memory usage, if a container goes past its memory limit it will be terminated. If your pod is managed by a Deployment, StatefulSet, DaemonSet, or another type of controller, then the controller spins up a replacement.

### Nodes

It is important to remember that you cannot set requests that are larger than resources provided by your nodes. For example, if you have a cluster of dual-core machines, a Pod with a request of 2.5 cores will never be scheduled! 

## Namespace settings

In an ideal world, Kubernetes’ Container settings would be good enough to take care of everything, but the world is a dark and terrible place. People can easily forget to set the resources, or a rogue team can set the requests and limits very high and take up more than their fair share of the cluster.

To prevent these scenarios, you can set up ResourceQuotas and LimitRanges at the Namespace level.

### ResourceQuotas

After creating Namespaces, you can lock them down using ResourceQuotas. ResourceQuotas are very powerful, but let’s just look at how you can use them to restrict CPU and Memory resource usage.

Looking at this example, you can see there are four sections. Configuring each of these sections is optional.

- requests.cpu is the maximum combined CPU requests in millicores for all the containers in the Namespace. In the above example, you can have 50 containers with 10m requests, five containers with 100m requests, or even one container with a 500m request. As long as the total requested CPU in the Namespace is less than 500m!
- requests.memory is the maximum combined Memory requests for all the containers in the Namespace. In the above example, you can have 50 containers with 2MiB requests, five containers with 20MiB CPU requests, or even a single container with a 100MiB request. As long as the total requested Memory in the Namespace is less than 100MiB!
- limits.cpu is the maximum combined CPU limits for all the containers in the Namespace. It’s just like requests.cpu but for the limit.
- limits.memory is the maximum combined Memory limits for all containers in the Namespace. It’s just like requests.memory but for the limit.

If you are using a production and development Namespace (in contrast to a Namespace per team or service), a common pattern is to put no quota on the production Namespace and strict quotas on the development Namespace. This allows production to take all the resources it needs in case of a spike in traffic.

### LimitRange

You can also create a LimitRange in your Namespace. Unlike a Quota, which looks at the Namespace as a whole, a LimitRange applies to an individual container. This can help prevent people from creating super tiny or super large containers inside the Namespace.

- The default section sets up the default limits for a container in a pod. If you set these values in the limitRange, any containers that don’t explicitly set these themselves will get assigned the default values.
- The defaultRequest section sets up the default requests for a container in a pod. If you set these values in the limitRange, any containers that don’t explicitly set these themselves will get assigned the default values.
- The max section will set up the maximum limits that a container in a Pod can set. The default section cannot be higher than this value. Likewise, limits set on a container cannot be higher than this value. It is important to note that if this value is set and the default section is not, any containers that don’t explicitly set these values themselves will get assigned the max values as the limit.
- The min section sets up the minimum Requests that a container in a Pod can set. The defaultRequest section cannot be lower than this value. Likewise, requests set on a container cannot be lower than this value either. It is important to note that if this value is set and the defaultRequest section is not, the min value becomes the defaultRequest value too.

## The lifecycle of a Kubernetes Pod

At the end of the day, these resources requests are used by the Kubernetes scheduler to run your workloads. It is important to understand how this works so you can tune your containers correctly.

Let’s say you want to run a Pod on your Cluster. Assuming the Pod specifications are valid, the Kubernetes scheduler will use round-robin load balancing to pick a Node to run your workload.

Note: The exception to this is if you use a nodeSelector or similar mechanism to force Kubernetes to schedule your Pod in a specific place. The resource checks still occur when you use a nodeSelector, but Kubernetes will only check nodes that have the required label.

Kubernetes then checks to see if the Node has enough resources to fulfill the resources requests on the Pod’s containers. If it doesn’t, it moves on to the next node.

If none of the Nodes in the system have resources left to fill the requests, then Pods go into a “pending” state. By using Kubernetes Engine features such as the Node Autoscaler, Kubernetes Engine can automatically detect this state and create more Nodes automatically. If there is excess capacity, the autoscaler can also scale down and remove Nodes to save you money!

But what about limits? As you know, limits can be higher than the requests. What if you have a Node where the sum of all the container Limits is actually higher than the resources available on the machine?

At this point, Kubernetes goes into something called an “overcommitted state.” Here is where things get interesting. Because CPU can be compressed, Kubernetes will make sure your containers get the CPU they requested and will throttle the rest. Memory cannot be compressed, so Kubernetes needs to start making decisions on what containers to terminate if the Node runs out of memory.

Let’s imagine a scenario where we have a machine that is running out of memory. What will Kubernetes do?

Note: The following is true for Kubernetes 1.9 and above. In previous versions, it uses a slightly different process. See this doc for an in-depth explanation.

Kubernetes looks for Pods that are using more resources than they requested. If your Pod’s containers have no requests, then by default they are using more than they requested, so these are prime candidates for termination. Other prime candidates are containers that have gone over their request but are still under their limit.

If Kubernetes finds multiple pods that have gone over their requests, it will then rank these by the Pod’s priority, and terminate the lowest priority pods first. If all the Pods have the same priority, Kubernetes terminates the Pod that’s the most over its request.

In very rare scenarios, Kubernetes might be forced to terminate Pods that are still within their requests. This can happen when critical system components, like the kubelet or docker, start taking more resources than were reserved for them.

## References

- https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-resource-requests-and-limits
