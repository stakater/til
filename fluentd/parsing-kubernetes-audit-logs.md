# Parsing kubernetes audit logs

Initially kubernetes audit logs source was

```fluentd
# Example:
# 2017-02-09T00:15:57.992775796Z AUDIT: id="90c73c7c-97d6-4b65-9461-f94606ff825f" ip="104.132.1.72" method="GET" user="kubecfg" as="<self>" asgroups="<lookup>" namespace="default" uri="/api/v1/namespaces/default/pods"
# 2017-02-09T00:15:57.993528822Z AUDIT: id="90c73c7c-97d6-4b65-9461-f94606ff825f" response="200"
<source>
  @type tail
  @id in_tail_kube_apiserver_audit
  multiline_flush_interval 5s
  path /var/log/kubernetes/kube-apiserver-audit.log
  pos_file /var/log/kube-apiserver-audit.log.pos
  tag kube-apiserver-audit
  <parse>
    @type multiline
    format_firstline /^\S+\s+AUDIT:/
    # Fields must be explicitly captured by name to be parsed into the record.
    # Fields may not always be present, and order may change, so this just looks
    # for a list of key="\"quoted\" value" pairs separated by spaces.
    # Unknown fields are ignored.
    # Note: We can't separate query/response lines as format1/format2 because
    #       they don't always come one after the other for a given query.
    format1 /^(?<time>\S+) AUDIT:(?: (?:id="(?<id>(?:[^"\\]|\\.)*)"|ip="(?<ip>(?:[^"\\]|\\.)*)"|method="(?<method>(?:[^"\\]|\\.)*)"|user="(?<user>(?:[^"\\]|\\.)*)"|groups="(?<groups>(?:[^"\\]|\\.)*)"|as="(?<as>(?:[^"\\]|\\.)*)"|asgroups="(?<asgroups>(?:[^"\\]|\\.)*)"|namespace="(?<namespace>(?:[^"\\]|\\.)*)"|uri="(?<uri>(?:[^"\\]|\\.)*)"|response="(?<response>(?:[^"\\]|\\.)*)"|\w+="(?:[^"\\]|\\.)*"))*/
    time_format %Y-%m-%dT%T.%L%Z
  </parse>
```

The audit logs changed its format so the above source wasn't parsing the logs. They were changed to json logs so updating the above to below should've fixed the issue

```fluentd
<source>
  @type tail
  @id in_tail_kube_apiserver_audit
  multiline_flush_interval 5s
  path /var/log/kubernetes/kube-apiserver-audit.log
  pos_file /var/log/kube-apiserver-audit.log.pos
  tag kube-apiserver-audit
  <parse>
    @type json
    keep_time_key true
    time_key timestamp
    time_format %Y-%m-%dT%T.%L%Z
  </parse>
</source>
```

But then elasticsearch started throwing the following error:

```json
    2018-10-04 13:46:39 +0000 [warn]: #0 dump an error event: error_class=Fluent::Plugin::ElasticsearchErrorHandler::ElasticsearchError error="400 - Rejected by Elasticsearch" location=nil tag="kube-apiserver-audit" time=2018-10-04 13:46:34.000000000 +0000 record={"kind"=>"Event", "apiVersion"=>"audit.k8s.io/v1beta1", "metadata"=>{"creationTimestamp"=>"2018-10-04T13:46:34Z"}, "level"=>"RequestResponse", "timestamp"=>"2018-10-04T13:46:34Z", "auditID"=>"0cc572e3-1a9d-42e6-b82e-6c24860a1610", "stage"=>"ResponseComplete", "requestURI"=>"/apis/rbac.authorization.k8s.io/v1beta1/clusterrolebindings", "verb"=>"create", "user"=>{"username"=>"kops", "groups"=>["system:masters", "system:authenticated"]}, "sourceIPs"=>["127.0.0.1"], "objectRef"=>{"resource"=>"clusterrolebindings", "name"=>"kubeadm:node-proxier", "apiGroup"=>"rbac.authorization.k8s.io", "apiVersion"=>"v1beta1"}, "responseStatus"=>{"metadata"=>{}, "status"=>"Failure", "message"=>"clusterrolebindings.rbac.authorization.k8s.io \"kubeadm:node-proxier\" already exists", "reason"=>"AlreadyExists", "details"=>{"name"=>"kubeadm:node-proxier", "group"=>"rbac.authorization.k8s.io", "kind"=>"clusterrolebindings"}, "code"=>409}, "requestObject"=>{"kind"=>"ClusterRoleBinding", "apiVersion"=>"rbac.authorization.k8s.io/v1beta1", "metadata"=>{"name"=>"kubeadm:node-proxier", "creationTimestamp"=>nil}, "subjects"=>[{"kind"=>"ServiceAccount", "name"=>"kube-proxy", "namespace"=>"kube-system"}], "roleRef"=>{"apiGroup"=>"rbac.authorization.k8s.io", "kind"=>"ClusterRole", "name"=>"system:node-proxier"}}, "responseObject"=>{"kind"=>"Status", "apiVersion"=>"v1", "metadata"=>{}, "status"=>"Failure", "message"=>"clusterrolebindings.rbac.authorization.k8s.io \"kubeadm:node-proxier\" already exists", "reason"=>"AlreadyExists", "details"=>{"name"=>"kubeadm:node-proxier", "group"=>"rbac.authorization.k8s.io", "kind"=>"clusterrolebindings"}, "code"=>409}}
```

Basically elasticsearch was rejecting the logs due to dynamic schema of these logs which wasn't supported in elasticsearch plugin as mentioned here: https://github.com/uken/fluent-plugin-elasticsearch/issues/452

A workaround mentioned in the link above was to modify the index template and disable indexing and mapping for dynamic fields one by one. Tried it with the following index template:

```json
    {
      "template": "logstash-*",
      "mappings": {
        "fluentd": {
          "dynamic_templates": [
            {
              "default_no_index": {
                "path_match": "^.*$",
                "path_unmatch": "^(@timestamp|auditID|level|stage|requestURI|sourceIPs|metadata|objectRef|user|verb)(\\..+)?$",
                "match_pattern": "regex",
                "mapping": {
                  "index": false,
                  "enabled": false
                }
              }
            }
          ]
        }
      }
    }
```

Now some logs were going through, and others still had the same issue. The reason was that every log had so many different fields and had different schemas. After some digging I found out that the logs had full blown kubernetes objects in them which caused the following issues:

- Not every log was successfully sent to elasticsearch due to dynamic schema. We would have to monitor each and every failed log and see which field is dynamic and add it to the mapping above in order to fix that
- The logs which went through, caused load on our cluster because of intensive parsing
- There were so many logs that they were filling the volumes at a rate of 50-100mb / 10 minutes

Found out that we had very verbose audit logs which we enabled ourselves by following this article: https://medium.com/@noqcks/kubernetes-audit-logging-introduction-464a34a53f6c

The only solution I can think of is that we should restrict its policy to just log `Metadata` instead of whole objects. That way we won't have to disable indexing of fields, or worry about space issues, and the logs would also be parsed since Metadata follows the same schema as mentioned in the article above like so:

```yaml
---
apiVersion: audit.k8s.io/v1beta1
kind: Policy
rules:
 — level: Metadata
 omitStages:
 — RequestReceived
```