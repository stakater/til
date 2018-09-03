# Access internet in a DIND container

We faced a problem where our build containers created by our pipelines weren't able to access network / internet.

## Solution

You need to enable IPTables and IPMasq for docker while setting up kops cluster.

Add the following to your cluster spec in the docker section (Create one if it doesn't exist):

```yaml
...
  docker:
    ipTables: true
    ipMasq: true
...
```

Update your cluster by running:

```bash
kops replace -f cluster.yaml
kops update cluster --yes
kops rolling-update cluster --yes
```

## Workaround

If you can't update your cluster, you can use this workaround. Add `--network=host` flag so that the build context and the containers use host's network.

*Note:* The flag `--network` is only supported with Docker API Version > 1.23. So make sure to change your Docker API version by using the environment variable `DOCKER_API_VERSION`.

There's another way to fix this as well. We can use the network of one of kubernetes pods which provides network to other pod containers. To do that, we can add `--network container:$(docker ps | grep $(hostname) | grep k8s_POD | cut -d " " -f1)` to docker commands.