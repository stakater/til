# Custom graph using prometheus api

Prometheus exposes its api over https in JSON format. The following link can be used for reference: 

```
https://prometheus.io/docs/prometheus/latest/querying/api/#expression-queries
```

I'm using `react-chartjs-2` for visually displaying data from prometheus. Current challenges include: 

* Understanding the `step` param in `query_range` type GET request. The doc is not very clear about this point, and having some trouble understanding it via results 

* Currently the support for changing the timespan for graph data was not added, and it by default shows the data of past 1 hour 

* For some queries the data become so large if we increase the time span that the graph starts to mix up things, and data is sandwiched. Looking for a way in the library to skip some labels, and to make things consistent. 