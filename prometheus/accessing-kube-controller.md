# How to Access Kube Controller In Prometheus

For Kube-Controllers to be accessed by Prometheus, you need a Service to expose the Pod/Process of Kube Controller and a Service Monitor for Prometheus to monitor that Service.

## Kubernetes

Verify that `kube-controller` pod is running in `kube-system` namespace, If yes, create a [Service](https://github.com/kahkhang/kube-linode/blob/master/manifests/prometheus/prometheus-k8s-service-monitor-kube-controller-manager.yaml) and a [Service Monitor](https://github.com/kahkhang/kube-linode/blob/master/manifests/prometheus/prometheus-k8s-service-monitor-kube-controller-manager.yaml) to monitor the kube-controller pods.

## Openshift

### Versions > 3.10

Firstly, check if `kube-controller` pod is running in `kube-system` namespace, If yes, you can simply create a [Service](https://github.com/openshift/cluster-monitoring-operator/blob/master/assets/prometheus-k8s/kube-controllers-service.yaml) and [Service Monitor](https://github.com/openshift/cluster-monitoring-operator/blob/master/assets/prometheus-k8s/service-monitor-kube-controllers.yaml) for it.

### Versions < 3.10

Firstly, check if `kube-controller` pod is running in `kube-system` namespace, if yes, follow the way provided in Versions > 3.10. If not, check the process running the `kube-controller` and see the port for it, and verify that it exposes metrics, if it does, you can write your own `Endpoints` object that has the IP addresses to the kube-controllers and deploy it manually and write a Service for it. You can do this by removing the `selector` from the [Service](https://github.com/openshift/cluster-monitoring-operator/blob/master/assets/prometheus-k8s/kube-controllers-service.yaml)