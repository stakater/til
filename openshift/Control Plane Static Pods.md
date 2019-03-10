# Control Plane Static Pods

Starting in OpenShift Container Platform 3.10, the deployment model for installing and operating the core control plane components changed. Prior to 3.10, the API server and the controller manager components ran as stand-alone host processes operated by systemd. In 3.10, these two components are moved to static pods operated by the kubelet.

For masters that have etcd co-located on the same host, etcd is also moved to static pods. RPM-based etcd is still supported on etcd hosts that are not also masters.

In addition, the node components openshift-sdn and openvswitch are now run using a DaemonSet instead of a systemd service.

![Control Plan Static Pods](../diagrams/Control Plane Static Pods.png)
