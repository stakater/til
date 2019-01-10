# Storing Last read log position in Fluentd

Fluentd looks for logs of pods and forwards it to Elasticsearch, and for different pods we have different configurations of fluentd. There is a parameter that can be used for fluentd to store the last read log settings.

## pos_file

This parameter is highly recommended. Fluentd will record the position it last read into this file.

`pos_file /var/log/td-agent/tmp/access.log.pos`

e.g.

```yaml
<source>
    @id fluentd-containers.log
    @type tail
    path /var/log/containers/*.log
    pos_file /var/log/fluentd-containers.log.pos
    time_format %Y-%m-%dT%H:%M:%S.%NZ
    tag raw.kubernetes.*
    format json
    read_from_head true
</source>
```

From the above config, when fluentd will parse these logs, it will save the last read position of container's log files in `/var/log/fluentd-containers.log.pos`. This path will be mounted from the host node and it will remain there even if fluentd goes down. And when fluentd starts again, it will know the last read position and will start reading logs after the last part so no log gets missed. You can access the file from the host node and read it. In my case, following is some part of it.

```txt
/var/log/containers/stakater-tools-sonatype-nexus-788949587f-lc6kq_tools_fmp-volume-permission2-e7547217c0727422b98dea3ae9a3a02c34560ae61c68115f413551dbad392b67.log    0000000000000000    00000000001ffd32
/var/log/containers/elasticsearch-operator-sysctl-ghw5s_default_sysctl-conf-3831729f39469d76c665bebcdf9db755c890bcc80ea0a455545c428559846c98.log    0000000000000000    0000000000182a97
/var/log/containers/pzoo-1_tools_zookeeper-9ead90159b287def3b824f97e73818a6c2be6edafc51b4b3e1cff963dfe7f14f.log    0000000000000000    00000000002ff5e2
/var/log/containers/test-fluentd-fluentd-elasticsearch-wh2gt_test-fluentd_test-fluentd-fluentd-elasticsearch-d8ded81aa88acf3e6c0893f7ab0b577e3fbc4c1513566fcbeceb13769bcf0eb9.log    0000000000001c19    000000000028ab00
```