# How to forcefully abort a Jenkins Job

## Problem
Sometimes jenkins job/pipelines get stuck and are not aborted by aborting them via the jenkins UI. 

## Solution

* Navigate to `Manage Jenkins` from the Jenkins Home page
* Scroll down and open `Script Console`
* In the script console run the following command: 

```
Jenkins.instance.getItemByFullName("JobName").getBuildByNumber(JobNumber).finish(hudson.model.Result.ABORTED, new java.io.IOException("Aborting build"));
```

Enter your Job Name inplace of `JobName` and Build number in place of `JobNumber` 

e.g.

```
Jenkins.instance.getItemByFullName("carbook/vehicles-service")
                .getBuildByNumber(47)
                .finish(
                        hudson.model.Result.ABORTED,
                        new java.io.IOException("Aborting build")
                );
```
