# Pushing assets to hosted raw reposiotry

First of all create a new nexus repository of type `raw`. A new method is added in `stakater-pipeline-library` in release version `v2.13.1` with the help of which we can upload assets to hosted nexus raw repositories. The method is present in file `ChartManager.groovy`, and is named `uploadToHostedNexusRawRepository`. 

The method takes the following params: 

```
    nexusUsername = Username for nexus repo
    nexusPassword = Password for nexus repo
    packagedChartLocation = Path for the newly created chart to push to nexus including file name. Must end with .tgz
    nexusURL = Base URL of the nexus
    nexusChartRepoName = Name of the nexus repository where we want to push our charts
```

`nexusUsername`: Should be read from Jenkins credentials and passed to the method
`nexusPassword`: Should be read from Jenkins credentials and passed to the method
`packagedChartLocation`: Path is already present in stage `Chart: Upload` in stakater-pipeline-library
`nexusURL`: Should be passed as a config variable
`nexusChartRepoName`: Should be passed as a config variable

## How it works

The whole process works in 3 steps:

    * First of all it uploads the newly created chart to nexus repo
    * Secondly it fetches all assets from nexus repo to re-generate the `index.yaml` file
    * Thirdly it regenerates index.yaml file, and pushes the updated file to nexus repo. 

## TODO

In order to add our nexus raw repo as a helm repo, we need to enable anonymous access in our nexus configuration. By default if we enable it then our full nexus server will become anonymously accessible. We need to create some roles in a way to make only this raw repo anonymously accessible.

