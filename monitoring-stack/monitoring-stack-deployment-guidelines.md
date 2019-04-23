# Monitoring Stack Deployment Guidelines

[Prometheus Operator](https://github.com/helm/charts/tree/master/stable/prometheus-operator) is being for the monitoring stack deployment. 

### Stack Deployment Guidelines

- Create a namespace:

```json
namespace-manifest.json

{ 
  "kind": "Namespace", 
  "apiVersion": "v1", 
  "metadata": { 
    "name": "monitoring", 
    "labels": { 
      "name": "monitoring" 
    } 
  } 
}
```
Use the command given below to create the namespace
```bash
$ sudo kubectl apply -f namespace-manifest.json
```

- Create a file to store the values that will override the default values of prometheus operator chart. Description of the variables are given in the Prometheus Operator github page, otherwise go through the github pages of each chart. 

```yaml
overridden_values.yaml

global:
  rbac:
    create: false
    pspEnabled: true

prometheusOperator:
  serviceAccount:
    name: tms-irti-service-account
  kubeletService:
    enabled: true
    # createCustomResource: false

commonLabels:
  expose: "true"

prometheus:
  service:
    labels:
      expose: true
    annotations:
      config.xposer.stakater.com/Domain: stakater.com
      config.xposer.stakater.com/IngressNameTemplate: '{{.Service}}-{{.Namespace}}'
      config.xposer.stakater.com/IngressURLPath: /
      config.xposer.stakater.com/IngressURLTemplate: '{{.Service}}.{{.Namespace}}.{{.Domain}}'
      xposer.stakater.com/annotations: |-
        kubernetes.io/ingress.class: external-ingress
        ingress.kubernetes.io/rewrite-target: /
        ingress.kubernetes.io/force-ssl-redirect: true

grafana:
  rbac:
    create: true
    namespaced: true
  ingress:
    enabled: "true"
    hosts:
      - grafana.test1.stakater.com
    annotations:
      kubernetes.io/ingress.class: "external-ingress"
      ingress.kubernetes.io/rewrite-target: "/"
      ingress.kubernetes.io/force-ssl-redirect: "true"

coreDns:
  service:
    port: 10055
    targetPort: 10055


```
Use the command given below to deploy the prometheus operator

`PMS`: Public helm charts monitoring stack
```bash
$ sudo helm install --name=pms  --namespace=monitoring . -f overridden_values.yaml
```


### Issue
In case of any issue go through the guidelines provided on this [link](https://github.com/helm/charts/tree/master/stable/prometheus-operator) 