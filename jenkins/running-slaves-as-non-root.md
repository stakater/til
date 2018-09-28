
# Running Slaves in Jenkins as non-root users

* When using an image with custom user, it is not able to access jenkins logs, as the logs are owned by the "jenkins" user with ID 10000, hence the pipeline hangs. Solution was to create a user with same id i.e. jenkins. 
* If using the Fabric8 maven plugin, they set maven.home to `/root/.m2` so it won't read the settings you mount on `/home/jenkins/.m2`, you will need to overwrite the `mavenOpts` (and maybe `javaOptions` if needed) by passing them in the mavenNode parameters.
* If you face permission issues with mounted mvnrepository permissions issue, Init containers are not possible with podTemplate of kubernetes-plugin, and specifying yaml through “yaml” key did not have any effect apparently (It should have according to documentation and examples). 
Workaround: Run pod in pipeline before the others, and own the directory if not already owned using chown (as root)