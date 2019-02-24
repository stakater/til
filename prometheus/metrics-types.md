# Metric Types

The Prometheus offer four core metric types.

A meter is uniquely identified by its name and dimensions (also called tags). Dimensions are a way of adding dimensions to metrics, so they can be sliced, diced, aggregated and compared. For example, we have a meter named http.requests with a tag uri. With this meter we could see the overall amount of HTTP requests, but also have the option to drill down and see the amount of HTTP requests for a specific URI.

## Counter

A counter is a cumulative metric that represents a single monotonically increasing counter whose value can only increase or be reset to zero on restart. For example, you can use a counter to represent the number of requests served, tasks completed, or errors.

Do not use a counter to expose a value that can decrease. For example, do not use a counter for the number of currently running processes; instead use a gauge.

Counters are a cumulative metric that represents a single numerical value that only ever goes up. They are typically used to count requests served, tasks completed, errors occurred, etc. Counters should not be used to expose current counts of items whose number can also go down, gauges are a better fit for this use case.

## Gauge

A gauge is a metric that represents a single numerical value that can arbitrarily go up and down.

Gauges are typically used for measured values like temperatures or current memory usage, but also "counts" that can go up and down, like the number of concurrent requests.

A gauge is a metric that represents a single numerical value that can arbitrarily go up and down. Gauges are typically used for measured values like current memory usage, but also “counts” that can go up and down, like the number of messages in a queue.

## Histogram

?

## Summary

?
