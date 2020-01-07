# Volume no mounting issue on master, data node

Elasticsearch had an issue that the volumes were not being attached to data, master nodes. The client node was not started. It was causing the cluster to not to be available in ready state. The client node was also not being started.

The correct way to start the cluster is in the way given below:

1. Data.
2. Master node.
3. Client node.

Fluentd was caching the data so no data was lost.
