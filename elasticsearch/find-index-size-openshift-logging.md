# Find Index Size in Openshift Logging Elasticsearch

3 Types of logs:
1. Operations Logs
2. Audit Logs
3. Project Logs

- Opearations have logs containing the logs of pods from namespaces with prefix `openshift-*` and have index `.operations`
- Audit logs have openshift audit logs and have index `.audit`
- Project logs have logs from the pods from all the namespaces except wiht prefix `openshift-*`


In order to find the indices size of the indices in elasticsearch,
- Go to Kibana -> Dev Tools
- On the left pane, remove all the request and write this:
```
GET _cat/indices?v
```
Results would be shown on the right column and `store.size` would br the total size of the index

TO delete:
```
DELETE /my-index-000001
```

Find the namespace distribution for a specific index:
```
POST /.operations.2020.11.16/_search?size=0
{
  "aggs": {
    "namespaces": {
       "terms": { "field": "kubernetes.namespace_name" } 
    }
  }
}
```