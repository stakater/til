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

In Openshift 3.7, we updated the image to v3.11, and added the above source in ConfigMap but it caused the fluentd pod to fall in CrashLoop Back Off, and in the logs it didn't gave any specific error. After some investigation, it turned out that the prometheus-conf was already added in the source and we were adding again the prometheus source so it went to fail state. When giving error, the configmap was 

```yaml
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
        hostname ${HOSTNAME}
      </labels>
    </source>


    ## ordered so that syslog always runs last...

    @include configs.d/openshift/input-pre-*.conf

    ...
```
And in `@include configs.d/openshift/input-pre-*.conf` the prometheus conf was also being included. So we fixed by editing the configmap to

```yaml
piVersion: v1
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
      hostname ${HOSTNAME}
     </labels>
    </source>


    ## ordered so that syslog always runs last...

    @include configs.d/openshift/input-pre-systemd.conf
```

We can update the configmap manually as even if the playbook is ran again, the configmap doesn't get replaced.