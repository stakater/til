# How to Handle Prometheus Operator Dependency

## Multiple Charts

You can have multiple charts, one for Prometheus Operator and other for Prometheus & related resources like Grafana, Kube-State-Metrics, etc. When you are going to deploy the Prometheus chart, always first upgrade or install the Prometheus Operator chart and then install the Prometheus chart.

## Single Chart

You can create a single umbrella chart and add Prometheus Operator, Prometheus and all other related resources in its dependencies.

In prometheus operator chart, you can create a new namespace with any name you like, I will be using `operators` and the deployment and ServiceAccount should be created in that `operators` namespace and change the clusterrolebinding and change the service account's namespace there as well.

The issue with single chart approach is you can't deploy two solutions e.g. ocp and qa, both with this dependency as in Prometheus Operator, we are creating a namespace, so if this chart is deployed twice, it will give error saying `operators` namespace already exists. So what you can do is, you can create a pre-req chart like `ocp` in our case, where we deploy all the system related apps and can add `operators` as dependency there and for other solutions like qa just deploy promethues related resources and not `prometheus-operator`