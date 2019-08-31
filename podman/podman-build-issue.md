# Podman Build Issue

## Issue

For images build from Podman version 1.4.4 It gives following error on Deploying to k8s cluster using containerd runtime

`Failed to pull image "docker.release.stakater.com:443/<image_name>": rpc error: code = Unknown desc = failed to pull and unpack image "docker.release.stakater.com:443/<image_name>": failed to unpack image on snapshotter overlayfs: failed to extract layer sha256:5b72799f7918f882eff3e2560d304f051109d9cbd7c2beacd6ac53ef4bbf5f3d: mount callback failed on /var/data/cripersistentstorage/tmpmounts/containerd-mount938212919: unexpected EOF: unknown`


- Podman builds/push images but they are not visible in nexus (same in public dockerhub). But when you pull it, it pulls the right image and gives the unpacking error.

- Podman builds/push images correctly (checked with a very basic Dockerfile) and runs fine on pull. (worked because only one `RUN echo` layer upon `alpine:3.9` image.)

## Reasons

Found [this](https://github.com/esa-esdl/jupyterhub-swarm/wiki/Outstanding-issues--(deployment-with-k8s)_) which states that it is a containerd problem.

Wrong Media type on layers by Buildah [issue opened here](https://github.com/containers/buildah/issues/1589) also suggested by IBM user on slack to wait for it to resolve for buildah.

## Tried steps:

- Visibilty issue resolved by specifying `--format docker` in podman build command

- Pushed same image to `docker.io` and `docker.release.stakater.com` to verfiy if it is not changed in the image repository (Both give the above error) (Conclusion: Nexus and dockerhub does not manipulate image)

- Tried with no `--disable-compression -D` flag but same error for both `docker.io` and `docker.release.stakater.com` (Conclusion: No issue with compression)

- Tried with `BUILDAH_FORMAT=docker` environment variable (mentioned in podman help). Gives this error:
`Error: failed to create containerd container: error unpacking image: failed to extract layer sha256:7684956cef99bb9d2cd2c020db914ddc1654792e366ab247e1d0db0ef679183c: mount callback failed on /var/data/cripersistentstorage/tmpmounts/containerd-mount367549648: unexpected EOF: unknown` (Conclusion: docker format on image repositories are correct)

- Tried with Ubuntu Image for `pipeline-tools` but same error. (Conclusion: No issue with centos vs. ubuntu)

- Tried building a golang application from local machine and pushed to `docker.io` and then deployed on IBM k8s but same error. (Conclusion: No issue specifically with our application Dockerfiles)

Podman Commands:
```
podman build --format docker -t <image>:<tag> . --network=host
podman push $(podman images -q | sed -n 1p) docker://<image>:<tag>
```