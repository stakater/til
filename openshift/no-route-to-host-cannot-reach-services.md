# No route to host (Pods unable to reach services)

We encountered an error in which pods on a new node were unable to talk to any other pod or service in openshift and they kept restarting with error `No route to host`.

The issue was with a newer version of iptables being installed on the node and it was incompatible with 3.7.62. The following solved the problem:

```bash
yum downgrade iptables-1.4.21-24.1.el7_5.x86_64 iptables-services-1.4.21-24.1.el7_5.x86_64
systemctl restart atomic-openshift-node
```

Explained here: https://access.redhat.com/solutions/3677791
