# Disabling read only api data volumes

Since kubernetes 1.9, they've ensured that runtime mounts volumes as read only. Meaning if we even try to modify the mounted configmap or secret inside a pod, it'll exit with a non 0 code that can lead to pod restarts. To revert this feature, we have to disable `ReadOnlyAPIDataVolumes` feature gate in the instance group like so:

```yaml
apiVersion: kops/v1alpha2
kind: InstanceGroup
metadata:
  name: nodes
spec:
  image: ami-02dea79d6a7f53d15
  ...
  kubelet:
    featureGates:
      ReadOnlyAPIDataVolumes: "false"
```