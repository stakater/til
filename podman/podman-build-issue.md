# Podman Build Issue

## Issue

For images build from Podman version 1.4.4 It gives following error on Deploying to k8s cluster using containerd runtime

`Failed to pull image "docker.release.stakater.com:443/<image_name>": rpc error: code = Unknown desc = failed to pull and unpack image "docker.release.stakater.com:443/<image_name>": failed to unpack image on snapshotter overlayfs: failed to extract layer sha256:5b72799f7918f882eff3e2560d304f051109d9cbd7c2beacd6ac53ef4bbf5f3d: mount callback failed on /var/data/cripersistentstorage/tmpmounts/containerd-mount938212919: unexpected EOF: unknown`


- On CentOS Podman builds/push images but they are not visible in nexus.(same in public dockerhub). But when you pull it, it pulls the right image and gives the unpacking error.

- On Ubuntu Podman builds/push images correctly (checked with a very basic Dockerfile) and runs fine on pull.

## Reason

Found [this](https://github.com/esa-esdl/jupyterhub-swarm/wiki/Outstanding-issues--(deployment-with-k8s)_) which states that it is a containerd problem.