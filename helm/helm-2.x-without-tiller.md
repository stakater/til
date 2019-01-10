# Helm 2.x without Tiller

## The problem with Tiller

- complicates RBAC since its not using the RBAC of the user or pod running the helm commands to read/write kubernetes resources - helm is talking to the remote tiller pod to do the work. If you are using a global tiller then thats often got something like the cluster-admin role which means anyone running helm commands effectively side steps RBAC completely! :).
- helm forces all releases to have a unique name within a tiller. This means if you have one global tiller then each release name must be globally unique across all namespaces. This leads to very long release names since they must typically append the namespace too. This means often service names are very different between environments as the release name is often included in the service name in many charts which breaks lots of the promise of using canonical service discovery in kubernetes.
We prefer to use same service names in every environment (development, testing, staging, production) - to minimise the amount of per-environment configuration that is required which avoids manual effort and reduces errors. e.g. refer to http://my-service/ in your app and it should just work in every namespace/environment your app is deployed in without wiring up special configuration.
- can cause lots of version conflicts between helm clients + tiller versions. We’ve seen this a lot in Jenkins X this year. e.g. a user has, say, helm 2.9 installed locally and installs Jenkins X. Then a build pod with helm 2.10 runs and barfs as the tiller and helm versions don’t match.

Solution to run helm 2.x without tiller is mentioned here:

https://jenkins-x.io/news/helm-without-tiller/
