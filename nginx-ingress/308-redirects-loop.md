# 308 Redirects when using SSL Termination on Load Balancer

## Problem
When using SSL termination on Load Balancer (nginx ingress annotation). The URLs show HTTP 308 Redirects loop. This error exists for
image >= quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.22.0

[Issue Thread](https://github.com/kubernetes/ingress-nginx/issues/1957#issuecomment-462826897)

## Solution

Create a configmap like below to fix the 308 redirect loops

```
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration-external
  namespace: control
data:
  use-forwarded-headers: "true"
```