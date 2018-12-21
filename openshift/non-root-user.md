# Non Root Contaners

## What Are Non-Root Containers?

By default, Docker containers are run as root users. This means that you can do whatever you want in your container, such as install system packages, edit configuration files, bind privilege ports, adjust permissions, create system users and groups, access networking information.

With a non-root container you can't do any of this . A non-root container should be configured for its main purpose, for example, run the Nginx server.

## Why Use A Non-Root Container?

Mainly because it is a best practise for security. If there is a container engine security issue, running the container as an unprivileged user will prevent the malicious code from scaling permissions on the host node. To learn more about Docker's security features, see this guide.

Another reason for using non-root containers is because some Kubernetes distributions force you to use them. For example Openshift, a Red Hat Kubernetes distribution. This platform runs whichever container you want with a random UUID, so unless the Docker image is prepared to work as a non-root user, it probably won't work due to permissions issues. 

Moreover, Openshift ignores the USER directive of the Dockerfile and launches the container with a random UUID.

## How To Create A Non-Root Container?

To explain how to build a non-root container image, we will use our Nginx non-root container and its Dockerfile.

```
FROM bitnami/minideb-extras:jessie-r22
LABEL maintainer "Bitnami <containers@bitnami.com>"

ENV BITNAMI_PKG_CHMOD="-R g+rwX"
...
RUN bitnami-pkg unpack nginx-1.12.2-0 --checksum cb54ea083954cddbd3d9a93eeae0b81247176235c966a7b5e70abc3c944d4339
...
USER 1001
ENTRYPOINT ["/app-entrypoint.sh"]
CMD ["nginx","-g","daemon off;"]
```

- The BITNAMI_PKG_CHMOD env var is used to define file permissions for the folders where we want to write, read or execute. The bitnami-pkg script reads this env var and performs the changes.
- The bitnami-pkg unpack nginx unpacks the Nginx files and changes the permissions as stated by the BITNAMI_PKG_CHMOD env var.

Up until this point, everything is running as the root user.

-Later, the USER 1001 directive switches the user from the default root to 1001. Although we specify the user 1001, keep in mind that this is not a special user. It might just be whatever UUID that doesn't match an existing user in the image. Moreover, Openshift ignores the USER directive of the Dockerfile and launches the container with a random UUID.

Because of this, the non-root images cannot have configuration specific to the user running the container. From this point to the end of the Dockerfile, everything is run by the 1001 user.

- Finally, the entrypoint is in charge of configure Nginx. It is worth mentioning that no nginx, www-data or similar user is created as the Nginx process will be running as the 1001 user. Also, because we are running the Nginx service as an unprivileged user we cannot bind the port 80, therefore we must configure the port 8080.

## Volumes In Kubernetes

Data persistence is configured using persistent volumes. Due to the fact that Kubernetes mounts these volumes with the root user as the owner, the non-root containers don't have permissions to write to the persistent directory.

The following are some things we can do to solve these permission issues:

- Use an init-container to change the permissions of the volume before mounting it in the non-root container. Example:

```
      spec:
        initContainers:
        - name: volume-permissions
          image: busybox
          command: ['sh', '-c', 'chmod -R g+rwX /bitnami']
          volumeMounts:
          - mountPath: /bitnami
            name: nginx-data
        containers:
        - image: bitnami/nginx:latest
          name: nginx
          volumeMounts:
          - mountPath: /bitnami
            name: nginx-data
```

- Use Pod Security Policies to specify the user ID and the FSGroup that will own the pod volumes. (Recommended)

```
      spec:
        securityContext:
          runAsUser: 1001
          fsGroup: 1001
        containers:
        - image: bitnami/nginx:latest
          name: nginx
          volumeMounts:
          - mountPath: /bitnami
            name: nginx-data
```

## Config Maps In Kubernetes

This is a very similar issue to the previous one. Mounting a config-map to a non-root container creates the file path with root permissions. Therefore, if the container tries to write something else in that path, it will result in a permissions error. The Pod Security Policies doesn't seem to work for configMaps so we will have to use an init-container to fix the permissions if necessary.
