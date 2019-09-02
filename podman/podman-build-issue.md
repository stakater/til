# Podman Build Issue

## Issues

### Issue # 1: Missing Image in Image Repository:
- Podman builds/push images but they are not visible in nexus (same in public dockerhub).

### Issue # 2: ImageErrPull Error while deploying in IKS
- For images build from Podman version 1.4.4+ It gives following error on Deploying to k8s cluster using containerd runtime
```
Failed to pull image "docker.release.stakater.com:443/<image_name>": rpc error: code = Unknown desc = failed to pull and unpack image "docker.release.stakater.com:443/<image_name>": failed to unpack image on snapshotter overlayfs: failed to extract layer sha256:5b72799f7918f882eff3e2560d304f051109d9cbd7c2beacd6ac53ef4bbf5f3d: mount callback failed on /var/data/cripersistentstorage/tmpmounts/containerd-mount938212919: unexpected EOF: unknown
```

## Issue # 1:

### Steps:
- In Nexus, every image had a field name format and `docker` was specified for images that were built/pushed using docker. Podman has a flag `--format`. So specifying `--format docker` in podman build command like the following solved the problem and now the image is shown in repository
```
podman build --format docker -t <image>:<tag> . --network=host
```
### Conclusion:
- Podman needs to specify the format of the image being built (default oci) so that image repositories can categorize the image via the format (docker or otherwise)


## Issue # 2:

### Step 1:
- Pushed same image to `docker.io` and `docker.release.stakater.com` to verfiy if the image is not being manipulated in the image repository. The error persisted
### Conclusion:
- Nexus and dockerhub does not manipulate image. Hence not a repository problem.



### Step 2:
- Built image with `--disable-compression -D` flag to check if unpacking error is related to compression and pushed to both `docker.io` and `docker.release.stakater.com`. The error persisted. 
```
podman build --format docker --disable-compression -t <image>:<tag> . --network=host
```
### Conclusion:
- No issue with compression



### Step 3:
- Built image with `BUILDAH_FORMAT=docker` environment variable (mentioned in podman help) to check if parameters are being passed to underlying buildah correctly (podman uses vendored buildah). The error persisted.
### Conclusion:
- No issue in way the buildah builds docker image.



### Step 4:
- Built the following simple Dockerfile on local machine (Ubuntu 18.06 + podman 1.5.0 installed via apt-get) and pushed to `docker.io` and pod was successfully deployed:
```
FROM alpine:3.9
RUN echo "Hello!"
```
### Conclusion:
- Images built on Ubuntu are working fine and images built on centos (pipeline-tools) is not.



### Step 5:
- Built the same Dockerfile on pipeline-tools container during the pipeline run and pod was successfully deployed.
### Conclusion:
- Minimal dockerfile has no issue during deployment in IKS.



### Step 6:
- Built a simple golang application from local machine and pushed to `docker.io` and then deployed on IBM k8s. The error persisted.
### Conclusion:
- No issue specifically with Dockerfiles of our applications.

## Reasons

Wrong Media type on layers inserted by Buildah [issue opened here](https://github.com/containers/buildah/issues/1589) also suggested by IBM user on slack to wait for it to resolve for buildah.

The reason a simple Dockerfile was able to run successfully was because it used only a simple `echo` command and not complex layers were inserted during the build process.


## Notes: 
Correct podman commands that should work:
```
podman build --format docker -t <image>:<tag> . --network=host
podman push $(podman images -q | sed -n 1p) docker://<image>:<tag>
```