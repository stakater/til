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
- name: ensure-aws-tags.service
  manifest: |
    Type=oneshot
    ExecStart=/usr/bin/docker run --net host --restart no quay.io/sergioballesteros/check-aws-tags
    ExecStartPost=/bin/systemctl restart kubelet.service
  requires:
  - docker.service
  roles:
  - Node
```