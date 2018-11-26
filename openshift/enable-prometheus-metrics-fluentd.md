# Enable Prometheus Metrics for Fluentd in Openshift

I am using Openshift 3.7 and I wanted to enable Prometheus plugin for fluentd so it sends metrics to Prometheus where I can have corresponding alerts.

For this, I want to update fluentd logging to 3.11, if I manually update image to 3.11 version and update the configmap to add following sources

```yaml
apiVersion: v1
data:
  fluent.conf: >
    # This file is the fluentd configuration entrypoint. Edit with care.


    ...
    ## sources

    # expose metrics in prometheus format
    
    <source>
      @type prometheus
      bind 0.0.0.0
      port 24231
      metrics_path /metrics
    </source>
    
    <source>
      @type prometheus_output_monitor
      interval 10
      <labels>
        hostname ${hostname}
      </labels>
    </source>

    ## ordered so that syslog always runs last...

   ...

```

it works fine, and I can check the prometheus metrics by curling the url `curl http://localhost:24231/metrics` from inside the pod, I am able to get the metrics.

This is a manual step and if you do this you can get prometheus metrics.

For auto steps, I am trying to update this through host file, in host file I added `openshift_logging_fluentd_image_version=v3.11` after `openshift_logging = true`, it updates the fluentd image to 3.11 in which the Prometheus endpoint is automatically exposed.

## Openshift 3.7

In Openshift 3.7, we updated the image to v3.11, and added the above source in ConfigMap but it caused the fluentd pod to fall in CrashLoop Back Off, and in the logs it didn't gave any specific error. After some investigation, it turned out that the prometheus-conf was already added in the source and we were adding again the prometheus source so it went to fail state. We need to remove our addition of sources in config map and check the default prometheus config which is at https://github.com/openshift/origin-aggregated-logging/blob/master/fluentd/configs.d/openshift/input-pre-prometheus-metrics.conf :

```yaml
<source>
  @type prometheus
  <ssl>
    enable true
    certificate_path "#{ENV['ES_CLIENT_CERT']}"
    private_key_path "#{ENV['ES_CLIENT_KEY']}"
    ca_path "#{ENV['ES_CA']}"
  </ssl>
</source>

<source>
  @type prometheus_monitor
  <labels>
    hostname ${hostname}
  </labels>
</source>

# excluding prometheus_tail_monitor
# since it leaks namespace/pod info
# via file paths

# This is considered experimental by the repo
<source>
  @type prometheus_output_monitor
  <labels>
    hostname ${hostname}
  </labels>
</source>
```

This is the default source of prometheus, it uses SSL and all the default parameters i.e. bind at `0.0.0.0` and port `24231` and path `/metrics`. You would have to `curl https://localhost:24231/metrics --insecure` and it can show you the metrics. But issue is that fluentd doesn't has a service, so how can prometheus scrape that endpoint as we are using Prometheus Operator and it uses Service Monitors to add targets from Services. I have found an additionalScrapeConfig in Prometheus which allows scraping data by giving config to it. I have used following way to configure it but I wasn't able to add it https://github.com/coreos/prometheus-operator/tree/master/example/additional-scrape-configs.  I have created following job config 

```yaml
- job_name: kubernetes-logging-fluentd-endpoints
  scrape_interval: 2m
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: https
  tls_config:
    ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    insecure_skip_verify: true
  kubernetes_sd_configs:
  - role: pod
    namespaces:
      names:
      - 'tools'
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    regex: fluentd-(.+)
```

and created a secret

```yaml
{
  "kind": "Secret",
  "apiVersion": "v1",
  "metadata": {
    "name": "additional-scrape-configs",
    "namespace": "namespace",  
  },
  "data": {
    "prometheus-additional.yaml": "LSBqb2JfbmFtZToga3ViZXJuZXRlcy1sb2dnaW5nLWZsdWVudGQtZW5kcG9pbnRzCiAgc2NyYXBlX2ludGVydmFsOiAybQogIHNjcmFwZV90aW1lb3V0OiAxMHMKICBtZXRyaWNzX3BhdGg6IC9tZXRyaWNzCiAgc2NoZW1lOiBodHRwcwogIHRsc19jb25maWc6CiAgICBjYV9maWxlOiAvdmFyL3J1bi9zZWNyZXRzL2t1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvY2EuY3J0CiAgICBpbnNlY3VyZV9za2lwX3ZlcmlmeTogdHJ1ZQogIGt1YmVybmV0ZXNfc2RfY29uZmlnczoKICAtIHJvbGU6IHBvZAogICAgbmFtZXNwYWNlczoKICAgICAgbmFtZXM6CiAgICAgIC0gJ3Rvb2xzJyAKICByZWxhYmVsX2NvbmZpZ3M6CiAgLSBzb3VyY2VfbGFiZWxzOiBbX19tZXRhX2t1YmVybmV0ZXNfcG9kX25hbWVdCiAgICBhY3Rpb246IHJlcGxhY2UKICAgIHJlZ2V4OiBmbHVlbnRkLSguKykKCg=="
  },
  "type": "Opaque"
}
```

But prometheus was still not able to add this in the configuration.