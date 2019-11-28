# Kubernetes dashboard adjust timeout 

Kubernetes dashboard by default has short session timeouts, around 10 minutes. Which can be very annoying since you need to 
login again after short periods of time. Following are the possible workarounds for this issue: 

## 1. Increase timeout:

Edit `kubernetes-dashboard` deployment, by default its deployed in namespace `kube-system`. And add an additional argument
`--token-ttl=3600` where 3600 represents number of seconds for the timeout.

`kubectl -n kube-system edit deployments kubernetes-dashboard`

```yaml
...

spec:
      containers:
      - args:
        - --auto-generate-certificates
        - --metric-client-check-period=300
        - --token-ttl=3600
...
```

Setting the value to 0 will disable the timeout.

## 2. Skip authentication:

Using the above workflow use the following arguments to disable authentication all together. 

```yaml
...

spec:
      containers:
      - args:
        - --auto-generate-certificates
        - --metric-client-check-period=300
        - --authentication-mode=basic
        - --token-ttl=0
        - --enable-skip-login
...
```
