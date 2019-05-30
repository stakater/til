# How to add dashboards in grafana.

I will explain how to add dashboard in grafana using sidecar dashboards. This guideline will be more skewed towards grafana deployments using [prometheus-operator helm chart](https://github.com/helm/charts/tree/master/stable/prometheus-operator) and [StakaterKubeHelmMonitoring](https://github.com/stakater/StakaterKubeHelmMonitoring). 


# Steps to do it

* First of all enable this flag:
```
grafana.sidecar.dashboards.enabled = true
```

* Add new dashboard using the `dashboard id` given on grafana site.

* Once dashboard is added in grafana, validate it whether it is working or not.

* Once everything is verified and validated, open the `share` tab of dashboard. Download the dashboard as a file.

* Now create a kubernetes config manifest file with `.yaml` extension and paste the content given below in that file:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: <name-of-dashboard>
  labels:
    grafana_dashboard: "1"
    app: test-grafana
    
    chart: prometheus-operator-5.0.13
    release: "release-name"
    heritage: "Tiller"
    expose: "true"
data:
   
  <name-of-dashboard>.json: |-
```

* Now copy the content of dashboard file and add it beneath the `<name-of-dashboard>.json: |-` statement.

* Validate the file structure. 

* Once eveything is verified and validated, run the command given below to create the new dashboard in grafana.
```bash
$ sudo kubectl apply -f <dashboard-name>.yaml -n <namespace-name>
```

* Details regarding how grafana uses the sidecar dashboard can be found in this [link](https://github.com/helm/charts/tree/master/stable/grafana#sidecar-for-dashboards)

