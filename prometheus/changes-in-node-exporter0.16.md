# Changes in Node Exporter 0.15.2 to 0.16

Node Exporter has had some breaking changes in v0.16. Previously our Grafana Nodes dashboard was working fine and showing all sorts of information about a node the image we used was `quay.io/prometheus/node-exporter:v0.15.2`, but when deploying in Openshift, we used the latest version of openshift image for node-exporter `openshift/prometheus-node-exporter:v0.16.0`, our Nodes dashboard broke and we were unable to fetch information from nodes.

After investigation, we found that in 0.16, there has been changes in the queries made to Prometheus. However, in `quay.io/prometheus/node-exporter:v0.16.0` these are not breaking changes i.e. the previous queries are also present, but in Openshift v0.16.0, these queries are removed.

## Changes to be made

| v0.15.2  Queryname            |    v0.16.0     Queryname                                                    |
|-----------------------|-------------------------------------------------------------------------------|
node_boot_time		  | node_boot_time_seconds |
node_cpu			  | node_cpu_seconds_total |
node_memory_SwapFree  | node_memory_SwapFree_bytes |
node_memory_MemTotal  | node_memory_MemTotal_bytes |
node_memory_Buffers   | node_memory_Buffers_bytes |
node_memory_Cached    | node_memory_Cached_bytes |
node_memory_MemFree   | node_memory_MemFree_bytes |
node_disk_bytes_read  | node_disk_read_bytes_total |
node_disk_bytes_written | node_disk_written_bytes_total |
node_disk_io_time_ms  | node_disk_io_time_seconds_total |
node_filesystem_size  | node_filesystem_size_bytes |
node_filesystem_free  | node_filesystem_free_bytes |
node_network_receive_bytes | node_network_receive_bytes_total |
node_network_transmit_bytes | node_network_transmit_bytes_total |