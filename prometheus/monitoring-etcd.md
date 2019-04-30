# ETCD-Monitoring

To configure Prometheus to scrape ETCD, first check if corresponding pods & services are running of etcd. If no, then create service for etcd pod. And add a label to that service you will be using in the service monitor. A sample service is given below.

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: stakater-kube-etcd  # Label on Service that ServiceMonitor will select
    jobLabel: kube-etcd
    release: monitoring
  name: kube-etcd
  namespace: kube-system
spec:
  clusterIP: None
  ports:
  - name: http-metrics
    port: 4001
    protocol: TCP
    targetPort: 4001
  selector:
    k8s-app: etcd-server   # Label on etcd pod, CHANGE_IT_ACCORDING_TO_YOUR_ETCD_POD
  sessionAffinity: None
  type: ClusterIP
```

Then create a ServiceMonitor for above Service, that Prometheus Operator should scrape. A sample ServiceMonitor is as follows:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:           # Labels used by Prometheus to select ServiceMonitor
    app: stakater-kube-etcd
    k8s-app: stakater-kube-etcd
    release: monitoring
  name: stakater-kube-etcd
  namespace: monitoring
spec:
  endpoints:
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    port: http-metrics
  jobLabel: k8s-app
  namespaceSelector:
    matchNames:
    - kube-system
  selector:
    matchLabels:
      app: stakater-kube-etcd
```

Now, etcd will be shown as targets in Prometheus, which you can see.

If it is working fine, then good, if not and it gives the erro `Get http://<master-ip>:4001/metrics: context deadline exceeded`,  you would have to open the metrics port on your master node, so open the respective port i.e. either 2379 or 4001, on the master, and check again, it will be able to successfully scrape it.

Now you can check in Prometheus queries, etcd metrics will be exposed based on your etcd version,

- etcd3: For etcd3 metrics, check https://github.com/etcd-io/etcd/blob/master/Documentation/metrics.md

- etcd2: For etcd3 metrics, check https://github.com/etcd-io/etcd/blob/master/Documentation/v2/metrics.md

If you have etcd3, you can configure this Grafana [dashboard](https://grafana.com/dashboards/3070).