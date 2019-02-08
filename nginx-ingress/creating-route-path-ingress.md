# Creating Ingress with Route based Path

I faced issue when creating Route Based path for ingress as it was not forwarding to the complete url, so I found that you need to add `ingress.kubernetes.io/rewrite-target: /` and  `ingress.kubernetes.io/add-base-url: "true"`. Below is a sample ingress:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: simple-ingress
  annotations:
    forecastle.stakater.com/appName: Grafana
    forecastle.stakater.com/expose: "true"
    forecastle.stakater.com/icon: https://raw.githubusercontent.com/stakater/ForecastleIcons/master/book.png
    ingress.kubernetes.io/force-ssl-redirect: "true"
    ingress.kubernetes.io/rewrite-target: /
    ingress.kubernetes.io/add-base-url: "true"
    kubernetes.io/ingress.class: external-ingress
spec:
  rules:
  - host: global.stakater.com
    http:
      paths:
      - path: /grafana
        backend:
          serviceName: grafana
          servicePort: http
```

This will create an ingress and can be accessed at `global.stakater.com/grafana`