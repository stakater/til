# IBM docker socket issue

With versions 1.11+, IBM Kubernetes Service (IKS) containerd replaces dockerd as the default container runtime. Previously the docker was mounted from `/var/run/docker.sock` but in IBM workers it is located at `/run/docker.sock`.