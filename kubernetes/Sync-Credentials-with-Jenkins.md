# Automating credential creation

To automate credential creation

- Add label to secret `credential.sync.jenkins.openshift.io=true`
- Add annotation to secret `jenkins.io/credentials-description : "description"`
- Install `kubernetes-credentials-provider:0.12.1` plugin in jenkins.

You can read more about it [here](https://jenkinsci.github.io/kubernetes-credentials-provider-plugin/examples/)
