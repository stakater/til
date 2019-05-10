# Monitoring Stack Deployment Guidelines

[Prometheus Operator](https://github.com/helm/charts/tree/master/stable/prometheus-operator) chart is being use for the monitoring stack deployment. 

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
ask irtiza for correct values
```

Use the command given below to deploy the prometheus operator

`PMS`: Public helm charts monitoring stack
```bash
$ sudo helm install --name=pms  --namespace=monitoring . -f overridden_values.yaml
```

### How to redeploy the monitoring stack

This section will provide guidelines on how to redeploy the monitoring stack

* Delete the monitoring stack deployment using the command given below:
```bash
$ sudo kubectl delete -f monitoringKubeHelmRelease.yaml -n monitoring
```

* Delete the respective CRDS given on [prometheus-operator](https://github.com/helm/charts/tree/master/stable/prometheus-operator) helm chart repository.

* Redeploy the monitoring stack using the command given below:
```bash
$ sudo kubectl apply -f monitoringKubeHelmRelease.yaml -n monitoring
```

* Open the monitoring namespace UI to check whether deployments, pods, services and ingresses are created or not.

* Ingress of Alertmanager, grafana and prometheus must be created.

* Also look at the logs of helm operator.

* Run the command given below to check chart's deployment status, it must be in deployed state:
```bash
$ sudo helm ls --namespace monitoring
```

### Issue
In case of any issue go through the guidelines provided on this [link](https://github.com/helm/charts/tree/master/stable/prometheus-operator)


