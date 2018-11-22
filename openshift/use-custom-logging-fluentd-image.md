# Use Custom Logging Image

I am using Openshift 3.7 and I wanted to enable Prometheus plugin for fluentd so it sends metrics to Prometheus where I can have corresponding alerts.

For this, I want to update fluentd logging to 3.11, if I manually update image to 3.11 version and update the configmap to

```yaml
apiVersion: v1
data:
  fluent.conf: >
    # This file is the fluentd configuration entrypoint. Edit with care.


    @include configs.d/openshift/system.conf


    # In each section below, pre- and post- includes don't include anything
    initially;

    # they exist to enable future additions to openshift conf as needed.


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

    @include configs.d/openshift/input-pre-*.conf

    @include configs.d/dynamic/input-docker-*.conf

    @include configs.d/dynamic/input-syslog-*.conf

    @include configs.d/openshift/input-post-*.conf

    ##


    <label @INGRESS>

    ## filters
      @include configs.d/openshift/filter-pre-*.conf
      @include configs.d/openshift/filter-retag-journal.conf
      @include configs.d/openshift/filter-k8s-meta.conf
      @include configs.d/openshift/filter-kibana-transform.conf
      @include configs.d/openshift/filter-k8s-flatten-hash.conf
      @include configs.d/openshift/filter-k8s-record-transform.conf
      @include configs.d/openshift/filter-syslog-record-transform.conf
      @include configs.d/openshift/filter-viaq-data-model.conf
      @include configs.d/openshift/filter-post-*.conf
    ##

    </label>


    <label @OUTPUT>

    ## matches
      @include configs.d/openshift/output-pre-*.conf
      @include configs.d/openshift/output-operations.conf
      @include configs.d/openshift/output-applications.conf
      # no post - applications.conf matches everything left
    ##

    </label>
  secure-forward.conf: |
    # <store>
    # @type secure_forward

    # self_hostname ${hostname}
    # shared_key <SECRET_STRING>

    # secure yes
    # enable_strict_verification yes

    # ca_cert_path /etc/fluent/keys/your_ca_cert
    # ca_private_key_path /etc/fluent/keys/your_private_key
      # for private CA secret key
    # ca_private_key_passphrase passphrase

    # <server>
      # or IP
    #   host server.fqdn.example.com
    #   port 24284
    # </server>
    # <server>
      # ip address to connect
    #   host 203.0.113.8
      # specify hostlabel for FQDN verification if ipaddress is used for host
    #   hostlabel server.fqdn.example.com
    # </server>
    # </store>
  throttle-config.yaml: |
    # Logging example fluentd throttling config file

    #example-project:
    #  read_lines_limit: 10
    #
    #.operations:
    #  read_lines_limit: 100
kind: ConfigMap
metadata:
  name: logging-fluentd
  namespace: logging
```

it works fine, and I can check the prometheus metrics by curling the url `curl http://localhost:24231/metrics` from inside the pod, I am able to get the metrics.

This is a manual step and if you do this you can get prometheus metrics.

For auto steps, I am trying to update this through host file, in host file I added `openshift_logging_image_version=v3.11` after `openshift_logging = true`, it updated the image but it causes another issue that for the side car oauth proxy containter of elasticsearch_master, it also gets the image version as 3.11, which should be 1.1.0, and I am unable to find a variable that can set that value. I have tried the following variables: 

- openshift_logging_elasticsearch_proxy_image=docker.io/openshift/oauth-proxy:v1.1.0
- openshift_logging_elasticsearch_proxy_image_version=v1.1.0

but they don't update it.