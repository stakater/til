
# Overview Purpose


##  Create credentials and secret text for user
Open the Jenkins jenkins dashboard. Add credentials for a user.

add following things:
- username
- password
- id (same as username)

and then creation a credential.


* create a secret text with following name: GithubToken, ask token from ali



## Update all the plugins

Go to Manage Jenkins -> Manage Plugins -> select all the plugins and update them


## Create a new Pipeline

Go to New Item -> Github Organization (name it stakater-lab)

* Add following things in the projects section:
-  filter by name (with regular expression) and behaviour should be Repositories.
with the following expression `.*nordmart.*`

* Also add advance clone behaviours.

* Add this regular expression `(PR-\d+|master)` in Automatic branch project triggering section.

Once all the configuration are done look at the Scan Organization log console to makesure that only pipeline for stacks run and no other pipeline run. 

Once process is completed, check whether projects have been added or not. If projects are added then their pipeline will run. Stop all those pipelines.

run each application pipeline individually, like catalog, product, web, gateway and inventory.


# TODOS
Create a Jenkinsfile in the root of the stakater-dev-apps. Move the scripts and Makefile out of the tools folder rename the targets by removing dev from them.

Create dry-run targets for other targets except the namespace creation target.

Now what will happend is that flux and related thing will be installed though the `nordmart-dev-tools` repo's jenkins file. When the repo run it will generate a command that can be retrieved from the Jenkins console. That command will be used to get the deploy key and that key will be added inside the `nordmart-dev-app` repository's deploy key. That deploy key can be added using the `stakater-user` account ask ali for it.
