# Problem: Fluentd is dropping logs

- Elasticsearch is not receiving all the logs from our pods
- Only a portion of the messages are arriving to Elasticsearch/Splunk

# Solution

- Check that Fluentd has enough resources allocated.

```
  resources:
    limits:
      cpu: 800m
      memory: 2Gi
```

- Fluentd can use up to 1000m CPU (it's a single-threaded process so cannot use more than one vCPU)
- Memory might also be a bottleneck, especially if increasing the CPU does not fix the issue

However usually if you are running low on memory the fluentd process will actually get oom-killed

Check if the process is being restarted with:

```
# oc exec $FLUENTPOD -- ps -ef
```

If the "PID" for fluentd is in the hundreds or thousands after a couple of hours or after a few stress tests, the process is probably restarting frequently, implying that there is not enough memory allocated to the pods.

- Update to the latest logging images to ensure all performance improvements are in place in the cluster. 
