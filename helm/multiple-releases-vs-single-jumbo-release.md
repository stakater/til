# Multiple Releases vs Single Jumbo Release

In case of a solution that comprises of multiple helm charts, Should we create a single jumbo 
release consisiting of all charts as its requirements or separate releases per chart? There are different approaches that we found to solve this problem

## Helm file

The route of an umbrella chart (One chart that pulled in a bunch of requirements) becames a bit of a pain to deal with since all your releases are bundled into one, If you ever got into a position where you needed to purge a release, you're a bit stuck. And secondly, if a single chart gets an error the whole release needs to be taken care of again. Especially since most of those charts aren't actually dependent on each other. Instead we found a way to deploy the charts as individual releases, but they are deployed 'together' using https://github.com/roboll/helmfile

## GitOps along with a single Requirements.yaml file

You can package, release and publish individual charts (components) but when it comes to deploy, use a single umbrella chart, and actually use a requirements.yaml per environment and gitops to control what and when is applied to each environment. A sample umbrella chart can be found at https://github.com/rawlingsj/environment-boltcarnelian-staging/tree/master/env. There’s some Jenkins X docs on this approach as well https://jenkins-x.io/about/features/#environments

