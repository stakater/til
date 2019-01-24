# How to push to Multiple Nexus

In order to push a single artifact to multiple nexus repositories: 

* Add another profile in maven-settings.xml (mounted as `jenkins-maven-settings` secret in our Pipelines)
* Update the URLs in that profile. 
* Add another `mvn deploy` in your pipeline but this time specify the profile as `mvn deploy -Pnexus-v2`. If you already ran mvn deploy on the first profile you can probably skip tests and install like `mvn deploy -Pnexus-v2 -Dmaven.test.skip=true -Dmaven.install.skip=true`.
* It is good to wrap this in a try catch block if you don't want the pipeline to fail when pushing to the 2nd nexus. 

In order to push docker images to multiple repositories:
* Update the docker config secret to add auths for all docker repositories. Then tag the image with repository URL prefix and push as usual.
