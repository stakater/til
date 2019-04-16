# Jenkins Deployment Strategies in Openshift

There are three main methods of deploying Jenkins in openshift.
- Use openshift jenkins image and deploy using jenkins template i.e `oc new-app jenkins-persistent`.
- Use openshift jenkins image and deploy using helm chart.
- Use public jenkins image and deploy using helm chart.

Strategy|Plugins|Credential|Pipelines|Script Approval|Issues|
---|---|---|---|---|---|
Use openshift jenkins image and deploy using jenkins template i.e `oc new-app jenkins-persistent`. | ? | Can be done if plugins are installed | Automated| No | Currently the available template doesn't take the optional parameter to specify plugins that you want to install but the image supports it so perhaps we can extend the default template and add the parameter|
Use openshift jenkins image and deploy using helm chart | Automated | Automated | Automated | No | The script approval doesn't work as there are compatibility issues between helm chart and openshift jenkins image. Also the helm chart adds volumes and init container that has nothing to do with the openshift jenkins image|
Use public jenkins image and deploy using helm chart| Automated | Automated | Automated | Automated | Right now this strategy doesn't allow `Login with Openshift`. We have found a plugin `Login with Openshift` in which you can manually enable the login but it needs more research to see if we can automate its configuration somehow |
