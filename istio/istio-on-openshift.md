# Istio Deployment on Openshift

## Overview

This document explains the guideline on how to deploy upstream istio on openshift. 

## Guideline
Deployment guideline is given on this [link](https://github.com/stakater/til/blob/master/istio/istio-on-openshift.md).

## Issues

1. I deployed istio on openshift but it wasn't able to inject sidecars inside application pods. After looking at the openshift [documentation](https://docs.openshift.com/container-platform/3.11/servicemesh-install/servicemesh-install.html#automatic-injection-comparison) i found out this:

```
The upstream Istio community installation automatically injects the sidecar to namespaces you have labeled.

Red Hat OpenShift Service Mesh does not automatically inject the sidecar to any namespaces, but requires you to specify the sidecar.istio.io/inject annotation as illustrated in the Automatic sidecar injection section.
```

I asked about the issue of deploying istio on openshift:

```
I believe using upstream is not possible, as it requires istio-cni to be deployed (I could be wrong though); and since istio-cni requires CNI plugin v0.3.x, this does not work with OCP/OKD 3.11

What I did was I deployed RH Service Mesh, though using maista-1.1 operator instead of one provided in documentation. If you want automatic sidecar injection to work, you need to set privileged SCC to serviceAccount that you are deploying with. Also keep in mind that in case you are using ovs-multitenant SDN network plugin, you need to make operators (and possibly istio) project globally accessible
```

For now it seems that the if we want to enable tracing we need to use the openshift's mesh service.

