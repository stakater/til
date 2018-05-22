# Logs Skipping Or Logs Missed or Logs Lost

On our OpenShift clusters we found out that at certain loads we started loosing logs and they don't appear in Splunk!

## Solution

First find out which logging driver is being used by the docker; and that you can do by running:

```
$ docker inspect -f '{{.HostConfig.LogConfig.Type}}' <CONTAINER>

journald
```

Then logon to any of the nodes and look the `journald` conf `/etc/systemd/journald.conf`

Also whenever log messages are suppressed then you will find log messages like

```
Suppressed 12965 messages from /system.slice/docker.service
```

which is indication that you will see logs not being shipped!


If its `journald` then the issue can be with rate limiting as default are following:

```
#RateLimitInterval=30s
#RateLimitBurst=1000
```

Update these settings and then restart the jounrald service:

```
sudo systemctl restart systemd-journald
```

If its k8s/openshift then you will need to update configs on each node and restart the service

Assuming you are running on k8s/openshift one can run a pod which logs e.g. 1000 messages per second:

```
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c,
            'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 0.001; done']
```

And then you can use this test app to verify that logs aren't skipping or suppressed any more.

For more info read:
- https://www.rootusers.com/how-to-change-log-rate-limiting-in-linux/
- https://github.com/moby/moby/issues/35653