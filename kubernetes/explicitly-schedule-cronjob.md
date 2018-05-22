# How to explicitly schedule a kubernetes cronjob 

## Problem
We have defined a cron job in kubernetes which runs after a specific time interval, but at some point we need to explicilty schedule the job.

## Solution
Create a `Job` from the existing `CronJob` using the following command.

```bash
  kubectl create job --from=cronjob/descheduler deschduler-manual -n kube-system

```
Here we run the deschduler cron job explicitly by created a Job named `deschduler-manual`

#### Note:

This syntax will work with `kubectl v1.10.1+`. It is tested on Kubernetes v1.8.7. 
