# Buildah vs. Podman

Buildah and Podman are two complementary Open-source projects which run without the daemon server.

More differences [here](https://podman.io/blogs/2018/10/31/podman-buildah-relationship.html)


## Why we chose to use Podman:

Because Podman can be used as an alias to docker command. All the container needs build/push are fulfilled by podman so buildah is not needed.