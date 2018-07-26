# kops with spot instances

## Problem

We were trying to add a new instance group with spot instances and we ran into race conditions where nodes started 
successfully but never got registered with master and cluster wasn't validated.

## Solution

The issue is that for some reason when we use spot instances the tags on instance appear late as compared to when kubelet
registers itself to master; and since tags are missing the master rejects it.

The solution is to restart kubelet once the tags have been available/added to the node.

More details can be found here:
https://github.com/kubernetes/kops/issues/3605

We need to add this hook cluster config:

```
apiVersion: kops/v1alpha2
kind: InstanceGroup
metadata:
  labels:
    kops.k8s.io/cluster: apps187.digitaldealer.devtest.aws.scania.com
  name: logging-nodes
spec:
  associatePublicIp: false
  image: ami-a61464df
  machineType: t2.2xlarge
  maxPrice: "0.4032"
  maxSize: 6
  minSize: 3
  nodeLabels:
    kops.k8s.io/instancegroup: logging-nodes
  taints:
  - dedicated=logging:NoSchedule
  role: Node
  rootVolumeSize: 50
  subnets:
  - eu-west-1a
  - eu-west-1b
  - eu-west-1c
  hooks:
    - name: disable-automatic-updates.service
      before:
      - update-engine.service
      manifest: |
        Type=oneshot
        ExecStartPre=/usr/bin/systemctl mask --now update-engine.service
        ExecStartPre=/usr/bin/systemctl mask --now locksmithd.service
        ExecStart=/usr/bin/systemctl reset-failed update-engine.service
    - name: ensure-aws-tags.service
      manifest: |
        Type=oneshot
        ExecStart=/usr/bin/docker run --net host --restart no quay.io/sergioballesteros/check-aws-tags
        ExecStartPost=/bin/systemctl restart kubelet.service
      requires:
      - docker.service
      roles:
      - Node
  kubelet:
    enforceNodeAllocatable: pods
    kubeReserved:
      cpu: 500m
      memory: 2Gi
    systemReserved:
      cpu: 500m
      memory: 2Gi
```