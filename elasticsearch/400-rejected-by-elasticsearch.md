# 400 Rejected By ElasticSearch

## Symptoms

- No logs reaching ES
- Fluentd logs full of this error; 400-Rejected by ElasticSearch

```
2019-01-14 14:54:29 +0000 [warn]: dump an error event: error_class=Fluent::Plugin::ElasticsearchErrorHandler::ElasticsearchError error="400 - Rejected by Elasticsearch" location=nil tag="kubernetes.var.log.containers.weave-net-lm7ft_kube-system_weave-npc-eeb3a810dec7336e0f644f33a00610c6f88505a748423682f1e870fe8d57acb0.log" time=2019-01-14 14:46:18.487371192 +0000 record={"log"=&gt;"INFO: 2019/01/14 14:46:18.487257 deleting entry 100.116.0.13 from weave-)=bW55XME{Tp;sBdO)~rU)q*m of b595e8f2-180a-11e9-b93a-020d118e7d9c\n", "stream"=&gt;"stderr", "docker"=&gt;{"container_id"=&gt;"eeb3a810dec7336e0f644f33a00610c6f88505a748423682f1e870fe8d57acb0"}, "kubernetes"=&gt;{"container_name"=&gt;"weave-npc", "namespace_name"=&gt;"kube-system", "pod_name"=&gt;"weave-net-lm7ft", "pod_id"=&gt;"ea31d669-ce6d-11e8-b93a-020d118e7d9c", "labels"=&gt;{"controller-revision-hash"=&gt;"3322957788", "name"=&gt;"weave-net", "pod-template-generation"=&gt;"3", "role_kubernetes_io/networking"=&gt;"1"}, "host"=&gt;"ip-10-4-151-52.eu-west-1.compute.internal", "master_url"=&gt;"https://100.64.0.1:443/api", "namespace_id"=&gt;"34f9843b-1a65-11e8-8255-06dc79b29256"}}
```

## Troubleshoot

To troubleshoot this error; the first step is to change the log level to debug:

```
    <match **>
      @id elasticsearch
      @type elasticsearch
      @log_level debug
      ...
```

Redeploy; and then look the logs again:

```
2019-01-14 15:33:18 +0000 [warn]: dump an error event: error_class=Fluent::Plugin::ElasticsearchErrorHandler::ElasticsearchError error="400 - Rejected by Elasticsearch [error type]: illegal_argument_exception [reason]: 'Rejecting mapping update to [logstash-2019.01.14] as the final mapping would have more than 1 type: [_doc, fluentd]'" location=nil tag="kube-controller-manager" time=2019-01-14 14:28:46.368055000 +0000 record={"severity"=&gt;"I", "pid"=&gt;"1", "source"=&gt;"wrap.go:42", "message"=&gt;"GET /healthz: (21.528\xC2\xB5s) 200 [[kube-probe/1.10] 127.0.0.1:40680]"}
```

```
Check if you have any old index template that might be specifying a different type and therefore causing a conflict.
```

