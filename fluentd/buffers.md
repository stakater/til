# Problem

- Apparently fluentd is running but its not sending logs to Splunk
- Fluentd looks healthy but doing nothing

# Solution

fluentd has buffers and if they fill up then fluentd will stop sending logs as it stuck

there can be different reasons why buffers fill up e.g.

- the reciever is down
- the reciever rejects the content

in such cases fluentd will store them in buffer so, that it can retry

on way to solve this:

- Cleanup this directory /var/lib/fluentd
- Then restart the fluentd pod
- The older logs will also show up; as fluentd keeps position
- Also keep in mind if you restart the node then you will loose logs and there will be a gap in logs

```
Seeems related to that /var/lib/fluentd gets full, 1,1Gb on stopped nodes

I guess this is where the buffer resides, its full of files;

buffer-output-es-config.output_tag.*.log

The size of this folder also seems to directly correspond to the current memory heap, seems to have al of this memory in buffer

*all of this buffer in memory

Our config in hosts/inventory file says the following;

openshift_logging_fluentd_buffer_queue_limit=1024
openshift_logging_fluentd_buffer_size_limit=1m
openshift_logging_fluentd_file_buffer_limit=1Gi

So both file_buffer_limit and possibly buffer_queue_limit could be related
```

Buffer will fill up if it can't push; we will see errors like this:

```
2018-11-27 14:04:17 +0100 [warn]: temporarily failed to flush the buffer. next_retry=2018-11-27 14:03:09 +0100 error_class="Fluent::ElasticsearchOutput::ConnectionFailure" error="Can not reach Elasticsearch cluster ({:host=>\"logging-es\", :port=>9200, :scheme=>\"https\", :user=>\"fluentd\", :password=>\"obfuscated\"})!" plugin_id="object:1942b58"
  2018-11-27 14:04:17 +0100 [warn]: suppressed same stacktrace
2018-11-27 14:04:52 +0100 [warn]: temporarily failed to flush the buffer. next_retry=2018-11-27 14:03:13 +0100 error_class="Fluent::ElasticsearchOutput::ConnectionFailure" error="Can not reach Elasticsearch cluster ({:host=>\"logging-es\", :port=>9200, :scheme=>\"https\", :user=>\"fluentd\", :password=>\"obfuscated\"})!" plugin_id="object:1942b58"
  2018-11-27 14:04:52 +0100 [warn]: suppressed same stacktrace
```
  
