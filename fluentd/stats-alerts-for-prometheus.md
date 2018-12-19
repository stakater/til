# Configuring Alerts in Prometheus for Fluentd

Fluentd is a core part of your logging stack, as it fetches logs and can send them to other applications where we can store and query those logs.

Fluentd itself exposes some metrics which Prometheus can scrape from. These metrics give very helpful information whether fluentd is working correctly or not and if there is any error in its working and we can further create alerts from these metrics.

We have used public helm chart for fluentd which is present at https://github.com/helm/charts/tree/master/stable/fluentd-elasticsearch and we have extended the default image of fluentd to add some plugins and configuration to support our workflow. There are quite some metrics that fluentd exposes about the output.

We have created three alerts on the output:

1. **max_over_time(fluentd_output_status_retry_wait[1m]) > 1000**

`fluentd_output_status_retry_wait` actually checks the current retry wait computed from last retry time and next retry time. This alert checks for the maximum retry wait in last 1 minute.

2. **rate(fluentd_output_status_retry_count[1m]) > 0.5**

This alert checks for the average retry count / sec in the last minute. If its value is greater than 0.5, it will get fired.

3. **max_over_time(fluentd_output_status_buffer_queue_length[1m]) > 500**

This alert will check the maximum buffer length in last 1 minute, which means that fluentd is unable to forward messages whether due to the output app like elasticsearch being down. So the buffer queue length will keep increasing. We have used 500, but its value can be increased depending upon the number of logs being generated.