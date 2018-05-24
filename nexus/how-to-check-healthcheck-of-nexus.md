# How to check health check of nexus3?

Nexus 3 has [metrics information](https://help.sonatype.com/repomanager3/configuration/support-features#SupportFeatures-Metrics) available in the administration support UI.

Additionally, Nexus 3.0.1 exposes the following REST endpoints for monitoring:

```
{host:port}/service/metrics/healthcheck
{host:port}/service/metrics/data
{host:port}/service/metrics/ping
{host:port}/service/metrics/threads
```

These require authentication, and the user must either be an administrator, or have the "nx-metrics-all" privilege.

```
curl -u admin:admin123 http://localhost:8081/service/metrics/ping
pong
```

```
curl -u admin:admin123 http://localhost:8081/service/metrics/healthcheck
{"deadlocks":{"healthy":true}}
```