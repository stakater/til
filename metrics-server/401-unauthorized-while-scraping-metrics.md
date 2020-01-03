# 401 Unauthorized error while scraping metrics

Recently, i came across the issue where metrics-server pod fails with the following error

`
I0101 10:43:52.667125       1 secure_serving.go:116] Serving securely on [::]:8443
E0101 10:44:52.672756       1 manager.go:111] unable to fully collect metrics: [unable to fully scrape metrics from source kubelet_summary:aks-rancher-31591068-0: unable to fetch metrics from Kubelet aks-rancher-31591068-0 (aks-rancher-31591068-0): request failed - "401 Unauthorized", response: "Unauthorized", unable to fully scrape metrics from source kubelet_summary:aks-rancher-31591068-1: unable to fetch metrics from Kubelet aks-rancher-31591068-1 (aks-rancher-31591068-1): request failed - "401 Unauthorized", response: "Unauthorized"]
`


Metrics-server now uses webhook authentication so for it to work properly, make sure Kubelet has webhook authentication enabled.
To do this, pass the following parameters to Kubelet, when creating the cluster:

```
anonymousAuth: false
authenticationTokenWebhook: true
authorizationMode: Webhook
```


Now, in case of `aks` we are using managed cluster so we don't have access to kubelet and the value of `authenticationTokenWebhook` is set to false. 
So, metrics-server won't work there. Although, aks deploys metrics-server by default as well.

Reference to the issue: https://github.com/Azure/AKS/issues/1087
