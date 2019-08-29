# Kaniko
[Github Repo](https://github.com/GoogleContainerTools/kaniko)

A container that is used to build docker images inside a container.

# Skopeo
[Github Repo](https://github.com/containers/skopeo)

Skopeo is a command line utility that performs operations (pull/push/delete/inspect) on container images and can communicate with image repositories. Skopeo can work with OCI images as well as the original Docker v2 images.

# Podman
[Github Repo](https://github.com/containers/libpod)

A rootless container/pod manager for k8s that is not dependant on server like docker and can build images without the need for mounting a socket on container. Uses buildah to build images

# Buildah
[Github Repo](https://github.com/containers/buildah)

Buildah is used to build images (docker/oci). Buildah commands are equivalent to Dockerfile commands. It can push/pull/build but cannot run containers.

# Buildah alongside Podman
[Podman Buildah relationship](https://github.com/containers/libpod#buildah-and-podman-relationship)

[More differences here](https://podman.io/blogs/2018/10/31/podman-buildah-relationship.html)

# Why use Podman ?

- Kaniko builds images as soon as the pod runs, but our pipeline library first runs all pods and then send commands one by one. Therefore incorporating Kaniko container will require a lot of changes.
- Podman is a standalone tool that has cli API almost similar to docker therefore podman is easy to incorporate and replace docker in our pipeline library.
- Podman is desirable as it is supported by RedHat, so moving to OpenShift will be smooth.
- buildah + skopeo can be used but it makes dependency on two tools. Podman provides all the operations in a single tool
- Podman uses vendored buildah
- Podman can run rootless and does not depends on a socket to be mounted from the host (More secure then docker) 