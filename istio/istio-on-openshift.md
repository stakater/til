# Istio Deployment on Openshift

## Overview

This document explains the guideline on how to deploy upstream istio on openshift. 

## Guideline
Will be added

## Issues

1. I deployed istio on openshift but it wasn't able to inject sidecars inside application pods. After looking at the openshift [documentation](https://docs.openshift.com/container-platform/3.11/servicemesh-install/servicemesh-install.html#automatic-injection-comparison) i found out this:

```
The upstream Istio community installation automatically injects the sidecar to namespaces you have labeled.

Red Hat OpenShift Service Mesh does not automatically inject the sidecar to any namespaces, but requires you to specify the sidecar.istio.io/inject annotation as illustrated in the Automatic sidecar injection section.
```

For now it seems that the if we want to enable tracing we need to use the openshift's mesh service.

