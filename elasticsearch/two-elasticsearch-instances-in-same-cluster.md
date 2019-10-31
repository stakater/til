# Two Elasticserch instances

## Need

Sometimes Applications that are hosted on our stacks need segregation (e.g. Same elasticsearch should not be used to view logs for an production/dev application that has infrastrucutre logs as well (control/delivery/monitoring/tracing/logging).) In order to separate apps and infrastructure logs two or more elasticsearch instances are required.

## Configuration

Two or more elasticsearch instances can be supported by separating the infrastructure and apps logs in fluentd configurations and then send them to their respective es instances.

In the following example 2 Elasticsearch instances have been setup in two different namespaces `logging` and `pliro-tools`.

- Rewrite tags by matching patterns of the namespace field by matching regex. as `<appname>.<original-tag>` and infrastructure logs as `<stakater>.<original-tag>`
- Match the tags and forward to respective Elasticsearch instances.

konfigurator-template.yaml:
```
# Rewrite tags as pliro.kubernetes.** for pliro and stakater.kubernetes.** for the rest of the logs
<match kubernetes.**>
    @type rewrite_tag_filter
    <rule>
    key $.kubernetes_namespace_name
    pattern /pliro-[a-z]+/
    tag stakater.${tag}
    invert true
    </rule>
    <rule>
    key $.kubernetes_namespace_name
    pattern /pliro-[a-z]+/
    tag $1.${tag}
    </rule>
</match>

# Send stakater.kubernetes.** to elasticsearch in logging stack
<match stakater.kubernetes.**>
    @id elasticsearch
    @type elasticsearch
    @log_level info
    include_tag_key true
    type_name _doc
    host "#{ENV['OUTPUT_HOST']}"
    port "#{ENV['OUTPUT_PORT']}"
    scheme "#{ENV['OUTPUT_SCHEME']}"
    ssl_version "#{ENV['OUTPUT_SSL_VERSION']}"
    ssl_verify false
    logstash_prefix "#{ENV['LOGSTASH_PREFIX']}"
    logstash_format true
    flush_interval 30s
    # Never wait longer than 5 minutes between retries.
    max_retry_wait 60
    # Disable the limit on the number of retries (retry forever).
    disable_retry_limit
    time_key timestamp
    reload_connections false
</match>

# Send the remaining pliro-(tools|dev|prod).kubernetes.** to elasticsearch in namespace pliro-tools  
<match *.kubernetes.**>  
    @id elasticsearch-pliro
    @type elasticsearch
    @log_level info
    include_tag_key true
    type_name _doc
    host "pliro-elasticsearch-data.pliro-tools"
    port "9200"
    scheme "#{ENV['OUTPUT_SCHEME']}"
    ssl_version "#{ENV['OUTPUT_SSL_VERSION']}"
    ssl_verify false
    logstash_prefix "#{ENV['LOGSTASH_PREFIX']}"
    logstash_format true
    flush_interval 30s
    # Never wait longer than 5 minutes between retries.
    max_retry_wait 60
    # Disable the limit on the number of retries (retry forever).
    disable_retry_limit
    time_key timestamp
    reload_connections false
</match>

```