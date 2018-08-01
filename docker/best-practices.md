# Docker Best Practices

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

## Always EXPOSE Important Ports

The EXPOSE instruction makes a port in the container available to the host system and other containers. While it is possible to specify that a port should be exposed with a docker run invocation, using the EXPOSE instruction in a Dockerfile makes it easier for both humans and software to use your image by explicitly declaring the ports your software needs to run:

- Exposed ports will show up under docker ps associated with containers created from your image
- Exposed ports will also be present in the metadata for your image returned by docker inspect
- Exposed ports will be linked when you link one container to another

## Use USER

By default docker containers run as root. A docker container running as root has full control of the host system. As docker matures, more secure default options may become available. For now, requiring root is dangerous for others and may not be available in all environments. Your image should use the USER instruction to specify a non-root user for containers to run as. If your software does not create its own user, you can create a user and group in the Dockerfile as follows:

```
RUN groupadd -r swuser -g 433 && \
useradd -u 431 -r -g swuser -d <homedir> -s /sbin/nologin -c "Docker image user" swuser && \
chown -R swuser:swuser <homedir>
```

### Reusing an Image with a Non-root User
The default user in a Dockerfile is the user of the parent image. For example, if your image is derived from an image that uses a non-root user swuser, then RUN commands in your Dockerfile will run as swuser.

If you need to run as root, you should change the user to root at the beginning of your Dockerfile then change back to the correct user with another USER instruction:

```
USER root
RUN yum install -y <some package>
USER swuser
```

## Always exec in Wrapper Scripts

Many images use wrapper scripts to do some setup before starting a process for the software being run. It is important that if your image uses such a script, that script should use exec so that the script’s process is replaced by your software. If you do not use exec, then signals sent by docker will go to your wrapper script instead of your software’s process. This is not what you want - as illustrated by the following example:

Say that you have a wrapper script that starts a process for a server of some kind. You start your container (using docker run -i), which runs the wrapper script, which in turn starts your process. Now say that you want to kill your container with CTRL+C. If your wrapper script used exec to start the server process, docker will send SIGINT to the server process, and everything will work as you expect. If you didn’t use exec in your wrapper script, docker will send SIGINT to the process for the wrapper script - and your process will keep running like nothing happened.

Also note that your process runs as PID 1 when running in a container. This means that if your main process terminates, the entire container is stopped, killing any child processes you may have launched from your PID 1 process.

## Use LABEL maintainer

The LABEL maintainer instruction sets the Author field of the image. This is useful for providing an email contact for your users if they have questions for you.

## Clean Temporary Files

All temporary files you create during the build process should be removed. This also includes any files added with the ADD command. For example, we strongly recommended that you run the yum clean command after performing yum install operations.

You can prevent the yum cache from ending up in an image layer by creating your RUN statement as follows:

```
RUN yum -y install mypackage && yum -y install myotherpackage && yum clean all -y
```

Note that if you instead write:

```
RUN yum -y install mypackage
RUN yum -y install myotherpackage && yum clean all -y
```

Then the first yum invocation leaves extra files in that layer, and these files cannot be removed when the yum clean operation is run later. The extra files are not visible in the final image, but they are present in the underlying layers.

The current Docker build process does not allow a command run in a later layer to shrink the space used by the image when something was removed in an earlier layer. However, this may change in the future. This means that if you perform an rm command in a later layer, although the files are hidden it does not reduce the overall size of the image to be downloaded. Therefore, as with the yum clean example, it is best to remove files in the same command that created them, where possible, so they do not end up written to a layer.

In addition, performing multiple commands in a single RUN statement reduces the number of layers in your image, which improves download and extraction time.

## References

- http://www.projectatomic.io/docs/docker-image-author-guidance/
