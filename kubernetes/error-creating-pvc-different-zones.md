# Error creating PVCs in different zones

Many times when you create PVCs in different zones with different pods, you will get the following error `Pod : Pod currently Pending. Reason: Unschedulable -> 0/n nodes are available: 1 Insufficient cpu, 1 node(s) had taints that the pod didn't tolerate, 2 node(s) had no available volume zone.​`  even though the PVC will be bound. 

I have faced this error mainly when deploying ElasticSearch cluster as we have to create different pods in different regions so we face this error.

This is caused by an issue in Kubernetes prior to version 1.12. The Kubernetes scheduler does not handle zone constraints when using dynamic provisioning of zone specific storage - such as for example EBS. First, the PersistenVolumeClaim is created and it is only then that the volume binding / dynamic provisioning occurs. This can lead to unexpected behavior where the PVC is created in a zone where there are actually no resources available for scheduling the pod.

Kubernetes 1.12 has released a new feature called Volume Binding Mode to control that behavior.

## Resolution

### Kubernetes 1.12 and later

It requires that the VolumeScheduling feature gate is enabled - which it is by default since 1.10. You need to add the field volumeBindingMode: WaitForFirstConsumer to the default storage class.

Here is an example:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: test-storage
volumeBindingMode: WaitForFirstConsumer
[...]
```

### Kubernetes < 1.12

The solution is to upgrade to Kubernetes 1.12.

If this is not possible, an obvious workaround is to use a single zone environment until access to Kubernetes 1.12 is possible.

Another workaround is to create single zone storage classes (one per Availability Zone). The Managed Master can be configured to use a specific storage class.


## Reference:

Found from https://support.cloudbees.com/hc/en-us/articles/360019540731-Autoscaling-issue-when-provisioning-Masters-in-Multi-AZ-Environment