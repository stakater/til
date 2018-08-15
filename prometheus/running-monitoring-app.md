# Running Monitoring Application

We want to deploy a [Java Spring Boot application](https://github.com/stakater/StakaterJavaMonitoringDemoApp). We have used [maven-centos](https://github.com/stakater/dockerfile-maven-centos) image as base image.

## Openshift

We want to run it as non-root as Openshift doesn't allow running a pod as root and creating directories. So in Dockerfile, we have set the USER as non-root as you can see below:

```Dockerfile
# Setting USER Id
ARG USER=1001520010

# Change to root user so that we can change ownership of the $HOME dir to our user
USER root
RUN chown -R $USER:0 /$HOME

# Change back to USER
USER $USER

# Creating Maven Package
RUN mvn clean -Duser.home=$HOME/application package -DskipTests=true

# Running Application
CMD mvn -Duser.home=$HOME/application spring-boot:run
```

In the above Dockerfile, you can see that we have explicitly given ownership to user `1001520010` of the $HOME folder and we will be packaging the maven application here so the user needs permissions to create sub-directories. We have used USER id = `1001520010` as in our cluster the minimum user we can set is `1001520001`.

Furthermore, in the deployment for the above application, in the SecurityContext, we have explicitly mentioned to `runAsUser: 1001520010` as if we don't mention this, the Cluster automatically choses an id and runs the pod as that user and the issue with this is that user will not have access to the $HOME folder so it will not be able to download dependencies.

## Vanilla Kubernetes

Similarly, as above, if you want to run monitoring app on Vanilla K8s, you can use the same Dockerfile as in the repo, and can even change the USER id to any other id e.g. 10001, and in the deployment manifest, you can remove the securityContext as it is not necessary.