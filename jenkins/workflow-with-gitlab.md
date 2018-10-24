# Workflow for Exhaustive MR With Gitlab

Jenkins jobs are triggered on every commit of every branch, but our pipeline for MR is very exhaustive as we deploy the PR artifacts in preview environment, so we don't want the pipeline to be built for every commit, so we have created a Workflow for it

- User creates a branch, commits code  -> pipeline gets triggered through push-event trigger -> We check in the pipeline if there is corresponding MR, if no, then return, if yes then complete the pipeline and its related stuff.

- User creates an MR -> pipeline triggered through Merge-Request event trigger -> checks in pipeline, corresponding MR found, run the complete pipeline and deploy artifacts in preview environment.

# Note:

The above workflow doesn't work for Multibranch pipeline, as it does not support Merge-Request event triggers. It returns error `Error 409 Merge Request Hook is not supported for this project`

## Reference for above issues

- https://github.com/jenkinsci/gitlab-plugin/issues/820
- https://github.com/jenkinsci/gitlab-plugin/issues/416#issuecomment-425185247

However, it works for freestyle jobs, So try with freestyle jobs.