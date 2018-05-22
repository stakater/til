# Create a pod which prints some text every second

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
            'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']
```

Sleep accepts decimal numbers so you can break it down this like:

1/2 of a second

```
sleep 0.5
```

1/100 of a second

```
sleep 0.01
```

So for a millisecond you would want

```
sleep 0.001
```

To fetch the logs, use the kubectl logs command, as follows:

```
$ kubectl logs counter
0: Mon Jan  1 00:00:00 UTC 2001
1: Mon Jan  1 00:00:01 UTC 2001
2: Mon Jan  1 00:00:02 UTC 2001
...
```