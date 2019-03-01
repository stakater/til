# Creating Grafana Dashboards

For creating a new dashboard, you can simply add a new variable from the setting of the dashboard, and set the field as by setting the query to `label_values(query, field_name)` e.g. if you want  a name variable set the query to `label_values(query, name)`.

Create a new panel and select graph, edit it,  And in metrics, select the data source to prometheus, and in A, write the Prometheus query, and in legend, select the value that you want  to differentiate with like `name` or `id`.

You can edit the different options given below in differnet tags to change values like showing in x-axis, showing total, avg or current value things.

## Creating a Table

When creating a table, edit it and in Metrics similarly select the query and change data source and query. But it will show all the labels as columns in the table so to prevent that you can use `min(kube_pod_info) by (pod)`, it will show you only one field then.