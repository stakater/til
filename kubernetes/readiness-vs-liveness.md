# Readiness vs Liveness Probes

## Liveness

- Enable Pod containers to recover from a broken state by being restarted
- Define periodic checks to determine if a Pod container is “alive” and if not, then it is killed and re-started
- Without Liveness probes, K8S is truly blind and unaware that our workloads may have silently failed or stopped working 

## Readiness

- Allow containers to indicate that they are “not ready” and therefore should temporarily not receive traffic
- Define periodic checks to determine if a Pod container is “ready” and if not, then it will stop receiving traffic until it safely can
- Enforced by removing endpoint IPs for Pods automatically so that they will not receive traffic for services that they support
- When a Pod transitions back to “readiness”, it will automatically resume receiving traffic without any intervention
- Without Readiness probes, K8S will send traffic to unready Pods and this can cause failures and unexpected results!
