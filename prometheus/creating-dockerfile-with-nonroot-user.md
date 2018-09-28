# Creating Dockerfile for non-root user

Whenever creating a Dockerfile, it is always recommended to not use a root user rather explicitly mention a user id in the Dockerfile as it can cause issues, specially when deploying on Openshift as OpenShift Platform runs containers using an arbitrarily assigned user ID. This provides additional security against processes escaping the container due to a container engine vulnerability and thereby achieving escalated permissions on the host node.

What happens is for each namespace/project Openshift assigns a range to run the pod from.

So for an image to support running as an arbitrary user, directories and files that may be written to by processes in the image should be owned by the root group and be read/writable by that group. Files to be executed should also have group execute permissions.

Adding the following to your Dockerfile sets the directory and file permissions to allow users in the root group to access them in the built image:

```yaml
RUN chgrp -R 0 /directory && \
    chmod -R g=u /directory
```


Because the container user is always a member of the root group, the container user can read and write these files. The root group does not have any special permissions (unlike the root user) so there are no security concerns with this arrangement. In addition, the processes running in the container must not listen on privileged ports (ports below 1024), since they are not running as a privileged user.

So a sample Dockerfile for running a maven application with non-root user can be 

```Dockerfile
ARG USER=1001
# Add code for application
ADD /application /$HOME/application/
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

References:
- https://docs.openshift.com/container-platform/3.10/creating_images/guidelines.html#openshift-specific-guidelines

- https://docs.okd.io/latest/creating_images/guidelines.html#use-env-vars

