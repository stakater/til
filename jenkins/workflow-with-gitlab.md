# Workflow for Exhaustive MR With Gitlab

## Work with Pipeline Jobs

Workflow is :

- User creates a branch, commits some code, the pipeline doesn't get triggered .

- User creates a MR, GitLab sends a Merge Request event, the pipeline gets the branch name as form that event payload and triggers that branch. the issue with this approach is that you cannot manually trigger the pipeline as the branch variable is not set. And if you set it as a parameter for the pipeline, it will override the variable coming from the event hook.

- So you can create two pipelines for each repo, one for auto triggering and other for manual triggering which you can use Multibranch pipeline for the manual pipeline.

### Issues with this approach

- One defined above that both Manual & Auto Triggering can be done at once.
- When the MR gets merged, the master does not get build as GitLab plugin does not build pipeline as the commit hash is same as that of MR. And it does not build, It  logs the error as `Last commit in Merge Request has already been built in build #BUILD_NO` in Jenkins logs. The GitHub plugin does not support it at their end. This [issue](https://github.com/jenkinsci/gitlab-plugin/issues/636) is still Open as of 30th October.

## Work with MultiBranch pipeline

Jenkins jobs are triggered on every commit of every branch, but our pipeline for MR is very exhaustive as we deploy the PR artifacts in preview environment, so we don't want the pipeline to be built for every commit, so we have created a Workflow for it

- User creates a branch, commits code  -> pipeline gets triggered through push-event trigger -> We check in the pipeline if there is corresponding MR, if no, then return, if yes then complete the pipeline and its related stuff.

- User creates an MR -> pipeline triggered through Merge-Request event trigger -> checks in pipeline, corresponding MR found, run the complete pipeline and deploy artifacts in preview environment.

### Note:

The above workflow doesn't work for Multibranch pipeline, as it does not support Merge-Request event triggers. It returns error `Error 409 Merge Request Hook is not supported for this project`

### Reference for above issues

- https://github.com/jenkinsci/gitlab-plugin/issues/820
- https://github.com/jenkinsci/gitlab-plugin/issues/416#issuecomment-425185247

However, it works for freestyle jobs, So try with freestyle jobs.