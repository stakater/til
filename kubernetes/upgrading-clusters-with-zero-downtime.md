# Upgrading your clusters with zero downtime

This is specific to Google Cloud Kubernetes Engine

When it comes to upgrading your cluster, there are two parts that both need to be updated: the masters and the nodes. The masters need to be updated first, and then the nodes can follow. 

## Upgrading the master with zero downtime

“Changing the master version can result in several minutes of control plane downtime. During that period you will be unable to edit this cluster.”

When the master goes down for the upgrade, deployments, services, etc. continue to work as expected. However, anything that requires the Kubernetes API stops working. This means kubectl stops working, applications that use the Kubernetes API to get information about the cluster stop working, and basically you can’t make any changes to the cluster while it is being upgraded.

So how do you update the master without incurring downtime?

So, therefore run highly available (multiple) masters with Kubernetes

## Upgrading nodes with zero downtime

When upgrading nodes, there are a few different strategies you can use. There are two I want to focus on: 

- Rolling update
- Migration with node pools

### Rolling Update

The simplest way to update your Kubernetes nodes is to use a rolling update.

A rolling update works in the following way. One by one, a node is drained and cordoned so that there are no more pods running on that node. Then the node is deleted, and a new node is created with the updated Kubernetes version. Once that node is up and running, the next node is updated. This goes on until all nodes are updated.

While it’s simple to perform a rolling update on Kubernetes Engine, it has a few drawbacks.

One drawback is that you get one less node of capacity in your cluster. This issue is easily solved by scaling up your node pool to add extra capacity, and then scaling it back down once the upgrade is finished.

The fully automated nature of the rolling update makes it easy to do, but you have less control over the process. It also takes time to roll back to the old version if there is a problem, as you have to stop the rolling update and then undo it.

### Migration with node pools

Instead of upgrading the “active” node pool as you would with a rolling update, you can create a fresh node pool, wait for all the nodes to be running, and then migrate workloads over one node at a time.

#### Step 1: Creating the new node pool

Create a new pool and then move the pods over

#### Step 2: Cordon & Drain the old pool

Now we need to move work to the new node pool. Let’s move over one node at a time in a rolling fashion.

First, cordon each of the old nodes. This will prevent new pods from being scheduled onto them.

   kubectl cordon <node_name>

Once all the old nodes are cordoned, pods can only be scheduled on the new nodes. This means you can start to remove pods from the old nodes, and Kubernetes automatically schedules them on the new nodes.

Warning: Make sure your pods are managed by a ReplicaSet, Deployment, StatefulSet, or something similar. Standalone pods won’t be rescheduled!

Run the following command to drain each node. This deletes all the pods on that node.

   kubectl drain <node_name> --force

After you drain a node, make sure the new pods are up and running before moving on to the next one.

If you have any issues during the migration, uncordon the old pool and then cordon and drain the new pool. The pods get rescheduled back to the old pool.

### Delete the old pool

Once all the pods are safely rescheduled, it is time to delete the old pool.

## References

- [Kubernetes best practices: upgrading your clusters with zero downtime](https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-upgrading-your-clusters-with-zero-downtime)
