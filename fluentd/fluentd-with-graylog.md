# Fluentd with graylog output

We use a custom version of openshift/ose-logging-fluentd:v3.11.0 which doesnt send logs to elasticsearch
For openshift logs, the easiest would be to use fluentd to collect all the logs directly
Some sources can be
# It contains all logs including apiserver, etcd and other containers / pods because in 3.11 everything is containerized and we don't need separate inputs for each of the kubernetes / openshift components
```<source>
  @type tail
  @id in_tail_container_logs
  path /var/log/containers/*.log
  pos_file /var/log/fluentd-containers.log.pos
  tag kubernetes.*
  read_from_head true
  <parse>
    @type json
    time_format %Y-%m-%dT%H:%M:%S.%NZ
  </parse>
</source>```
And a useful filter is the kubernetes_metadata which you can use after source
```<filter kubernetes.**>
  @type kubernetes_metadata
  @id filter_kube_metadata
</filter>```
It will add all the necessary metadata to your logs in order to distinguish between pods, namespaces and container level logs.
Then you can use the gelf plugin to send to graylog and then use the native forwarder plugin to send to logstash as well if desired
https://github.com/bodhi-space/fluent-plugin-gelf-hs
The GELF plugin will send logs in json format to the graylog server. There is a good article on that
https://blog.insiderattack.net/bunyan-json-logs-with-fluentd-and-graylog-187a23b49540

I think the following should be tried:
1. Setup the latest version of fluentd with bare-minimum config
```
<source>
  @type tail
  @id in_tail_container_logs
  path /var/log/containers/*.log
  pos_file /var/log/fluentd-containers.log.pos
  tag kubernetes.*
  read_from_head true
  <parse>
    @type json
    time_format %Y-%m-%dT%H:%M:%S.%NZ
  </parse>
</source>

<filter kubernetes.**>
  @type kubernetes_metadata
  @id filter_kube_metadata
</filter>

<match **>
  @type stdout
</match>

```
2. Then try to use the GELF plugin instead of the stdout plugin as the destination
