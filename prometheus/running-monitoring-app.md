# Running Monitoring Application

We want to deploy a [Java Spring Boot application](https://github.com/stakater/StakaterJavaMonitoringDemoApp). We have used [maven-centos](https://github.com/stakater/dockerfile-maven-centos) image as base image.

## Openshift

We want to run it as non-root as Openshift doesn't allow running a pod as root and creating directories. So in Dockerfile, we have set the USER as non-root as you can see below:

```Dockerfile
# Setting USER Id
ARG USER=1001
# Add code for application
ADD [--chown=$USER:root] /application /$HOME/application/

WORKDIR $HOME/application

# Exposing 8080 port as we want application to run on port 8080
EXPOSE 8080

# Change to root user so that we can change ownership of the $HOME dir to root group
USER root

# Creating Maven Package
RUN mvn clean -Duser.home=$HOME/application package -DskipTests=true

# Change the group to root group
RUN chgrp -R 0 /$HOME && \
    chmod -R g=u /$HOME

# Change back to USER
USER $USER

# Running Application
CMD mvn -Duser.home=$HOME/application spring-boot:run
```

In the above Dockerfile, we have used the `USER 1001` but when running on OpenShift, it will automatically choose an arbitrary user id based on the constraints of your cluster. So it will give error that the user does not has enough permissions to  create a directory. To overcome this, we have used the groups.

For an image to support running as an arbitrary user, directories and files that may be written to by processes in the image should be owned by the root group and be read/writable by that group. Files to be executed should also have group execute permissions.\\

Adding the following to your Dockerfile sets the directory and file permissions to allow users in the root group to access them in the built image:

```yaml
# Change the group to root group
RUN chgrp -R 0 /$HOME && \
    chmod -R g=u /$HOME
```

Because the container user is always a member of the root group, the container user can read and write these files. The root group does not have any special permissions (unlike the root user) so there are no security concerns with this arrangement.

## Vanilla Kubernetes

Similarly, as above, if you want to run monitoring app on Vanilla K8s, you can use the same Dockerfile as in the repo, and can even change the USER id to any other id.