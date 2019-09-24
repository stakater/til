# Logging in containerd (IBM)

In IKS (IBM Kubernetes Service) `Containerd` replaces `Docker`. The directory structure for containerd is different then docker therefore logs for containers are located in a different direction. Moreover, the Logging format is also different for containerd.

In docker logs reside in: `/var/log/docker/` whereas for containerd: `/var/log/containers` which contains symbolic links to the actual containers running under `/var/data/containers/`.

Therefore following should be done for logging 

1. Use latest fluentd-elasticsearch chart which incorporates the containerd format and directory logging structure.
2. Mount `/var/data` on fluent-elasticsearch pods so that the logs can be reached by fluentd.