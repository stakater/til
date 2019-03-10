# Control Plane Static Pods

From: https://docs.openshift.com/container-platform/3.10/architecture/infrastructure_components/kubernetes_infrastructure.html#control-plane-static-pods

Starting in OpenShift Container Platform 3.10, the deployment model for installing and operating the core control plane components changed. Prior to 3.10, the API server and the controller manager components ran as stand-alone host processes operated by systemd. In 3.10, these two components are moved to static pods operated by the kubelet.

For masters that have etcd co-located on the same host, etcd is also moved to static pods. RPM-based etcd is still supported on etcd hosts that are not also masters.

In addition, the node components openshift-sdn and openvswitch are now run using a DaemonSet instead of a systemd service.

![Diagram](Control Plane Static Pods.png)

Even with control plane components running as static pods, master hosts still source their configuration from the /etc/origin/master/master-config.yaml file, as described in the [Master and Node Configuration](https://docs.openshift.com/container-platform/3.10/install_config/master_node_configuration.html#install-config-master-node-configuration) topic.

