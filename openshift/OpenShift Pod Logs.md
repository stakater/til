# OpenShift Pod/Container Logs

Container logs can be viewed using the OpenShift cli.

```
oc logs <pod_name>
```

- Add option "-p" print the logs for the previous instance of the container in a pod if it exists.
- Add option "-f" to stream the logs

Logs are saved to the nodes disk where the container/pod is running and is located at:

```
/var/lib/docker/containers/$CONTAINER_ID/$CONTAINER_ID-json.log
```

Generate a list of the largest files to confirm that the log files are using a large percent of the disk space.

```
# find /var/lib/docker/ -name "*.log" -exec ls -sh {} \; | sort -n -r | head -20
# du -aSh /var/lib/docker/ | sort -n -r | head -n 10
```
