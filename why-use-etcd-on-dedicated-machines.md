# etcd is being run on a dedicated set of machines for the following reasons:

- The etcd lifecycle is not tied to Kubernetes. We should be able to upgrade etcd independently of Kubernetes.
- Scaling out etcd is different than scaling out the Kubernetes Control Plane.
- Prevent other applications from taking up resources (CPU, Memory, I/O) required by etcd.
