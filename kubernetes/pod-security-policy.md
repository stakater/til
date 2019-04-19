# Pod Security Policy

[Pod Security Policies](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) enable fine-grained authorization of pod creation and updates. It is a cluster-level resource that controls security sensitive aspects of the pod specification. It is an admission controller.

## How to enable it?

Open the `/etc/kubernetes/manifests/kube-apiserver.yaml` file. Append `PodSecurityPolicy` keyword in the `--enable-admission-plugins` element of `spec.containers.command` list.

Doing this in existing system will lock it down, which means existing pods can be recreated and new pods can also not be created.
