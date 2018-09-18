# Prometheus Reading Data from Node Exporter Vs Kubelet

Prometheus can read data about a cluster from Node Exporters and from Kubelet, depending which dashboard you use in Grafana. There are some dashboards which use node-exporters to gather data and some use metrics from CAdvisors running in Kubelets. You can see that in the url of the nodes that it shows in the dashboard. See the below image:

![Alt text](img/node_info.png?raw=true "Figure 1: Node URL")

You can see the port that it shows, Usually node-exporters run on 9100 whereas Kubelets run 4194 or 10250 or 10255 based on the k8s version you are using.

## What difference does it make

Node Exporters and Kubelets have different set of queries  that Prometheus makes to scrape data. So if you have two dashboards, for showing usage of your cluster, you might get slightly different values. We have used two dashboards

### Kubernetes Cluster Usage: Using Kubelets

Below is the dashboard using kubelets, and showing information about a node, its memory usage and other details. This dashboard is showing 70% memory has been used for this node.

![Alt text](img/kubelet-dashboard.png?raw=true "Figure 2: Dashboard using Kubelets")

### Nodes: Using Node Exporter

Below is the dashboard using node exporters, and showing details about a node. However, this dashboard is showing 60% memory being used.

![Alt text](img/nodeexporter-dashboard.png?raw=true "Figure 3: Dashboard using NodeExporter")

## Queries

Their corresponding Prometheus queries are different thats why this difference between the numbers.

### Kubelet

 The kubelet dashboard uses `sum (container_memory_working_set_bytes{id="/",instance=~"^$Node$"}) / sum (machine_memory_bytes{instance=~"^$Node$"}) * 100` query to gather information about the memory usage.

### Node Exporter

Whereas, Node Exporter one uses `((node_memory_MemTotal_bytes{instance="$server"} - node_memory_MemFree_bytes{instance="$server"}  - node_memory_Buffers_bytes{instance="$server"} - node_memory_Cached_bytes{instance="$server"}) / node_memory_MemTotal_bytes{instance="$server"}) * 100` for Memory consumption.

You can also see such difference when you increase or decrease memory of a node, Node Exporter will be able to cater to the change instantly, and show you the new memory of the node but dashboard using kubelet might not get the change and you might have to restart the node manually to restart the kubelet. And then it would show the latest values.