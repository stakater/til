# Kubernetes Operations (kops)

[kops](https://github.com/kubernetes/kops) is a tool for Production Grade K8s Installation, Upgrades, and Management.
It can be used to create new cluster on different cloud providers and generate the output in different formats including terraform, cloud formation etc for state management.
You can do a lot of configuration via kops including specifying kubernetes version, kube api server, creation of different instance groups e.g master, app-nodes, tool-nodes etc, specifying details like instance type, user data etc. You can also specify files and hooks (systemd units) that you would want to run on the different servers.
