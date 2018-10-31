# Workflow for Exhaustive MR With Gitlab

We have tried multiple ways of creating pipelines for our GitLab repos.

## Work with Pipeline Jobs (Recommended)

Workflow is :

- User creates a branch, commits some code, the pipeline doesn't get triggered.

- User creates a MR, GitLab sends a Merge Request event, the pipeline gets the branch name from that event payload and triggers that branch. The issue with this is that you cannot manually trigger the pipeline as the branch variable is not set as it sets through event trigger payload. And if you create a parameter for the pipeline, the issue is it will always override the variable coming from the event hook. So we are unable to invoke both steps at once i.e. the manual triggering and auto-triggering.

- So we can create two pipelines for each repo, one for auto triggering and other for manual triggering for which you can use Multibranch pipeline for the manual pipeline.

- Another issue with this approach is when the MR gets merged, the master does not get build as GitLab plugin does not build pipeline as the commit hash is same as that of MR. And it does not build, It  logs the error as `Last commit in Merge Request has already been built in build #BUILD_NO` in Jenkins logs. The Gitlab plugin does not support it at their end. This [issue](https://github.com/jenkinsci/gitlab-plugin/issues/636) is still Open as of 30th October.

- There are two potential ways to solve this problem, one is to modify the plugin, and deploy the plugin manually which is not a  suitable way as you will have to maintain it in the future yourself. Other approach can be to create another job for the repo, which will be triggered whenever a push hook event is triggered in the master branch.

- So in the end, you will have 3 jobs per repo:

1. The job which gets triggered whenever a MR is created or any commits are made in that MR branch.

2. The job which will be triggered when the MR gets merged into master, or any commits are made directly into master.

3. The job which will be triggered manually, this will be Multibranch pipeline, where you would have to scan it, and it will show you all the branches and you can run them manually.

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