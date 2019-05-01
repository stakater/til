# ETCD-Monitoring

## 1. Check if Pods Running

Check in the kube-system namespace that etcd pods are running. There will be static etcd pods running. Exec in those pods and curl the metrics port and path to confirm that etcd is exporting metrics. Mostly it will be one of these:

- `curl localhost:2379/metrics`
- `curl localhost:4001/metrics`
- `curl --cacert "/etc/etcd/ca.crt" --cert "/etc/etcd/peer.crt" --key "/etc/etcd/peer.key" https://master-ip:2379/metrics`

You should be able to see the metrics in etcd pod.

## 2. Check if etcd Service created

Check if corresponding services are running of etcd. If yes, then skip this step. If no, then create service for etcd pod. And add a label to that service you will be using in the service monitor. A sample service is given below.

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

## 3. Create ServiceMonitor

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
    scheme: https
  jobLabel: k8s-app
  namespaceSelector:
    matchNames:
    - kube-system
  selector:
    matchLabels:
      app: stakater-kube-etcd    # Label on Service of etcd
```

Now, etcd will be shown as targets in Prometheus, which you can see.

## 4. Troubleshooting

You should be able to see the etcd in Prometheus targets, If you see errors, following are some common errors that you might see:

### Get http://<master-ip>:4001/metrics context deadline exceeded

If it gives the error `Get http://<master-ip>:4001/metrics: context deadline exceeded`,  you would have to open the metrics port on your master node, so open the respective port i.e. either 2379 or 4001, on the master, and check again, it will be able to successfully scrape it.

### x509:certificate signed by unknown authority

If it gives `x509:certificate signed by unknown authority`, It means you would have to configure Prometheus to scrape etcd using certificates. Exec in the etcd pod and copy the content of the certs, you would need the content of `/etc/etcd/ca/ca.crt` and `/etc/etcd/ca/ca.key` and `/etc/etcd/etcd-client.crt` credential files. The path might be different based on your volume mounts. Copy above files and save them on your local machine. Now we need to create a secret with above files, run the following command. 

```sh
cat <<-EOF > etcd-cert-secret.yaml
apiVersion: v1
data:
  etcd-client-ca.crt: "$(cat ca.crt | base64 --wrap=0)"
  etcd-client.crt: "$(cat peer.crt | base64 --wrap=0)"
  etcd-client.key: "$(cat peer.key | base64 --wrap=0)"
kind: Secret
metadata:
  name: <secret-name>
  namespace: <prometheus-namespace>
type: Opaque
EOF
```

and then apply the above file i.e. etcd-cert-secret.yaml in your prometheus namespace which will create a secret with the content of the certificates used to call the etcd pods.

Now in Promethues, you will need to mount this additional secret, so in Prometheus Spec, add the following line
```yaml
secrets:
- <secret-name created above>
```

the above line will mount the secret in prometheus pod at `/etc/prometheus/secrets/` and in your ServiceMonitor, reference the cert-files so your ServiceMonitor will be:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:           # Labels used by Prometheus to select ServiceMonitor
    app: stakater-kube-etcd
    k8s-app: stakater-kube-etcd
  name: stakater-kube-etcd
  namespace: <service-monitor-namespace>
spec:
  endpoints:
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    honorLabels: true
    interval: 30s
    path: /metrics
    port: http-metrics
    scheme: https
    tlsConfig:
      caFile: "/etc/prometheus/secrets/<secret-name created above>/etcd-client-ca.crt"
      certFile: "/etc/prometheus/secrets/<secret-name created above>/etcd-client.crt"
      keyFile: "/etc/prometheus/secrets/<secret-name created above>/etcd-client.key"
  jobLabel: k8s-app
  namespaceSelector:
    matchNames:
    - kube-system
  selector:
    matchLabels:
      app: stakater-kube-etcd    # Label on Service of etcd
```

Prometheus will be able to scrape the targets successfully now.

## 5. Metrics

Now you can check in Prometheus queries, etcd metrics will be exposed based on your etcd version,

- etcd3: For etcd3 metrics, check https://github.com/etcd-io/etcd/blob/master/Documentation/metrics.md

- etcd2: For etcd3 metrics, check https://github.com/etcd-io/etcd/blob/master/Documentation/v2/metrics.md

## 6. Grafana

If you have etcd version 3, you can configure this Grafana [dashboard](https://grafana.com/dashboards/3070). Note it will not work on etcd version 2.