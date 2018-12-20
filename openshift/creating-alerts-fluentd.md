# Creating Alerts in Fluentd in Openshift 3.7

We are using Openshift 3.7 and we wanted to enable Prometheus plugin for fluentd so it sends metrics to Prometheus where we can have corresponding alerts.

For this, we need to update the fluentd image to 3.11 which supports exposing prometheus endpoint and metrics by default. By updating the image, fluentd exposes the metrics on port 24231 and prometheus can easily scrape it.

As we are using Prometheus Operator to deploy Prometheus, so for prometheus to be able to scrape metrics we need a service for fluentd pods so prometheus can scrape using service monitor. But with default logging playbook, it doesn't create a service for fluentd rather creates only pods. So, we need to create a service of fluentd in logging namespace. Then we will have to create a service monitor for that service so Prometheus can add fluentd as its target.

Once it is setup, fluentd will be sending metrics to prometheus, so we will now be needing to create alerts based on the metrics. For that, we have created 3 alerts:

1. **max_over_time(fluentd_output_status_retry_wait[1m]) > 100**

`fluentd_output_status_retry_wait` actually checks the current retry wait computed from last retry time and next retry time. This alert checks for the maximum retry wait in last 1 minute and if it is greater than 100 so it will fire the alert.

2. **rate(fluentd_output_status_retry_count[1m]) > 0.025**

This alert checks for the average retry count / sec in the last minute. If its value is greater than 0.025, it will get fired.

3. **max_over_time(fluentd_output_status_buffer_queue_length[1m]) > 20**

This alert will check the maximum buffer length in last 1 minute, which means that fluentd is unable to forward messages whether due to the output app like elasticsearch being down. So the buffer queue length will keep increasing. We have used 20, but its value can be decreased/ increased depending upon the number of logs being generated.

These alerts will help us know if there is any internal issue with fluentd, e.g. if the output stream gets down or somehow fluentd is unable to send alerts to output, so we can take corresponding actions.