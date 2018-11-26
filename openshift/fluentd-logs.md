# Fluentd Logs not showing in Console

The fluentd default image in Openshift doesn't log its logs to the console, therefore if you check the logs of the fluentd pod, you will not be able to see the error in the logs. If you check in the Dockerfile of the fluentd pod, you can see that they send the logs to another file rather than sending them to console. 

```yaml
Fluentd logs have been redirected to: /var/log/fluentd/fluentd.log
If you want to print out the logs, use command:
oc exec <pod_name> -- logs
```

But if the fluentd pod itself gets stuck in CrashLoop Back Off, you can't see its logs from the console.