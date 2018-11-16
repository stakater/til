# Splunk queries

Number of events per namespace:

```
index="openshift" | stats count by kubernetes.namespace_name
```

Avg size of events per namespace:

```
index="openshift" | eval esize=len(_raw) | stats avg(esize) by kubernetes.namespace_name
```

Exact size of events:

```
index="openshift" | eval esize=len(_raw) | stats sum(esize) by kubernetes.namespace_name
```
