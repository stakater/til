# Why jx? And why not fabric8?

Red Hat decided that Fabric8 platform was gonna be the upstream OSS project for a hosted SaaS that includes some 
CI/CD plus eclipse Che & a new OSS issue tracker - all running on a hosted OpenShift Online. Since we left it’s 
looking like the upstream Fabric8 platform project is now dead - all effort is now on the SaaS fork for openshift.io. 
The focus of Jenkins X is really around CI/CD on any kubernetes cluster - on premise or any cloud hosted cluster. 
In addition Jenkins X adds Preview Environments, GitOps promotion through environments and helm & skaffold support. 
Another big change is Jenkins X focusses on integration with best of breed technologies 
(github issues / JIRA / VS Code / IDEA etc) or tools like helm /skaffold / Anchore etc whereas openshift.io is 
creating a brand new issue tracker & focusing on Eclipse Che as the IDE

All the underlying files for the build packs - then on each app for docker, skaffold, helm & Jenkins pipelines are 
all in git so can be tweaked if required