# Download of images is very slow?

Pods are failing to start with following errors in logs:

Errors like:

```
Failed to pull image "docker.tools.tools178.k8syard.com/dd/contract:1.4.12": rpc error: code = Unknown desc = error pulling image configuration: received unexpected HTTP status: 504 Gateway Time-out
Error syncing pod
```

```
Failed to pull image "docker.tools.tools178.k8syard.com/dd/plan:1.4.0": rpc error: code = Unknown desc = unexpected EOF
Error syncing pod
```

stuck on downloading:
```
pulling image "stakater/ingressmonitorcontroller:1.0.7"
```

stuck on downloading:
```
pulling image "registry.opensource.zalan.do/teapot/external-dns:v0.4.5"
```

Seems to can't download from our own nexus3, dockerhub and even zalando registry

## Solution

```
The root cause was networking messed up due to changing OS version and docker versions.

Why it was slow all across was due to nodes taking too much time download from our nexus
```