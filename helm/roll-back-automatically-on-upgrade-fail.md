# Rollback Automatically on Upgrade Failure

In pipeline, usually we use `helm upgrade --install` but in case if an error occurs, there is no way we can roll back automatically so this can cause issues in pipelines.

So to overcome this, we can use the following command:

```bash
helm upgrade --install --wait --force --recreate-pods $(RELEASE_NAME) $(CHART_NAME) --namespace $(CHART_NAME) || (helm rollback --force --recreate-pods $(RELEASE_NAME) 0 ; exit 1)
```

This command will upgrade the chart and if any error occurs, it will rollback to the previous revision and exit with code 1 meaning pipeline will fail. The 0 specified above means the last release.

But whenever we rollback to any version, it creates a new revision i.e. if I am curently on revision 10, and now I upgraded and an error occurred which means the revision 11 failed, so now when the above command will rollback to previous version i.e. 10, so for doing that it will create a new revision i.e. 12. So the deployed revision will be 12 now.