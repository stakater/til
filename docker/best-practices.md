# Best Practices

Some docker specific best practices.

## Avoid Multiple Processes

We recommend that you do not start multiple services, such as a database and SSHD, inside one container. This is not necessary because containers are lightweight and can be easily linked together for orchestrating multiple processes.

## Place Instructions in the Proper Order

Docker reads the Dockerfile and runs the instructions from top to bottom. Every instruction that is successfully executed creates a layer which can be reused the next time this or another image is built. It is very important to place instructions that will rarely change at the top of your Dockerfile. Doing so ensures the next builds of the same image are very fast because the cache is not invalidated by upper layer changes.

For example, if you are working on a Dockerfile that contains an ADD command to install a file you are iterating on, and a RUN command to yum install a package, it is best to put the ADD command last:

```
FROM foo
RUN yum -y install mypackage && yum clean all -y
ADD myfile /test/myfile
```

This way each time you edit myfile and rerun docker build, the system reuses the cached layer for the yum command and only generates the new layer for the ADD operation.

If instead you wrote the Dockerfile as:

```
FROM foo
ADD myfile /test/myfile
RUN yum -y install mypackage && yum clean all -y
```

Then each time you changed myfile and reran docker build, the ADD operation would invalidate the RUN layer cache, so the yum operation would need to be rerun as well.

## Set Environment Variables

It is good practice to set environment variables with the ENV instruction. One example is to set the version of your project. This makes it easy for people to find the version without looking at the Dockerfile. Another example is advertising a path on the system that could be used by another process, such as JAVA_HOME.

## Avoid Default Passwords

It is best to avoid setting default passwords. Many people will extend the image and forget to remove or change the default password. This can lead to security issues if a user in production is assigned a well-known password. Passwords should be configurable using an environment variable instead.

If you do choose to set a default password, ensure that an appropriate warning message is displayed when the container is started. The message should inform the user of the value of the default password and explain how to change it, such as what environment variable to set.

## Avoid SSHD

It is best to avoid running SSHD in your image. You can use the docker exec command to access containers that are running on the local host. 

Installing and running SSHD in your image opens up additional vectors for attack and requirements for security patching.

## Use Volumes for Persistent Data

Images should use a Docker volume for persistent data. This way Kubernetes mounts the network storage to the node running the container, and if the container moves to a new node the storage is reattached to that node. By using the volume for all persistent storage needs, the content is preserved even if the container is restarted or moved. If your image writes data to arbitrary locations within the container, that content might not be preserved.

All data that needs to be preserved even after the container is destroyed must be written to a volume. With Docker 1.5, there will be a readonly flag for containers which can be used to strictly enforce good practices about not writing data to ephemeral storage in a container. Designing your image around that capability now will make it easier to take advantage of it later.

Furthermore, explicitly defining volumes in your Dockerfile makes it easy for consumers of the image to understand what volumes they need to define when running your image.
