# Running Kube-State-Metrics

For running Kube-State-Metrics and accessing them via Prometheus, you can check the manifests from [Kubernetes Manifests](https://github.com/kubernetes/kube-state-metrics/tree/master/kubernetes) . Or if deploying through helm, check the repository [chart-openshift-kube-state-metrics](https://github.com/stakater/chart-openshift-kube-state-metrics) for openshift or for Vanilla Kubernetes [chart-kube-state-metrics](https://github.com/stakater/chart-kube-state-metrics) .

## TroubleShooting

Check the logs of the Kube-State-Metrics container, if it gives error regarding 

- **listen tcp 0.0.0.0:81: bind: permission denied**
    
    You might not be using the official image, check the deployment of the openshift chart mentioned above, you can see two ports are being defined there, 8080 and 8081, use that deployment.


- **Failed to list *v2alpha1.CronJob: the server could not find the requested resource**
    
    CronJobs were added to Kubernetes in 1.8, and if your cluster is older than that you would have to manually specify the collectors and remove CronJobs from there. You can see in the deployment in above openshift chart, we have passed the collector flag to the kube-state-metrics container and have specified the resources we want and have removed CronJob from there

```
    containers:
      - name: kube-state-metrics
        image: quay.io/coreos/kube-state-metrics:v1.3.1
        args:
        - --collectors=daemonsets, pods, deployments, limitranges, nodes, replicasets, replicationcontrollers, resourcequotas, services, jobs, statefulsets, persistentvolumeclaims, namespaces, horizontalpodautoscalers
        ports:
        - name: http-metrics
          containerPort: 8080
        - name: telemetry
          containerPort: 8081
        readinessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 20
          timeoutSeconds: 5
```

- **context deadline exceeded** in Prometheus targets
    
    Increase the scraping time interval in service monitors to be `2m` and remove the addon-resizer container from the deployment.
