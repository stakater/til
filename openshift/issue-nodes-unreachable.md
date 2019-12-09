# Nodes Unreachable issue on creating Openshift


## Problem

When Openshift 3.11 is created using terraform, it fails to deploy and gives the following error:

```
null_resource.node_config (remote-exec): PLAY [all] *********************************************************************
null_resource.node_config (remote-exec): TASK [Gathering Facts] *********************************************************
null_resource.node_config (remote-exec): fatal: [infra2.openshift.local]: UNREACHABLE! => {"changed": false, "msg": "SSH Error: data could not be sent to remote host \"infra2.openshift.local\". Make sure this host can be reached over ssh", "unreachable": true}
null_resource.node_config (remote-exec): fatal: [node-databas1.openshift.local]: UNREACHABLE! => {"changed": false, "msg": "SSH Error: data could not be sent to remote host \"node-databas1.openshift.local\". Make sure this host can be reached over ssh", "unreachable": true}
null_resource.node_config (remote-exec): fatal: [infra1.openshift.local]: UNREACHABLE! => {"changed": false, "msg": "SSH Error: data could not be sent to remote host \"infra1.openshift.local\". Make sure this host can be reached over ssh", "unreachable": true}
null_resource.node_config (remote-exec): fatal: [master1.openshift.local]: UNREACHABLE! => {"changed": false, "msg": "SSH Error: data could not be sent to remote host \"master1.openshift.local\". Make sure this host can be reached over ssh", "unreachable": true}
null_resource.node_config (remote-exec): fatal: [node-verksamhet1.openshift.local]: UNREACHABLE! => {"changed": false, "msg": "SSH Error: data could not be sent to remote host \"node-verksamhet1.openshift.local\". Make sure this host can be reached over ssh", "unreachable": true}
null_resource.node_config (remote-exec): 	to retry, use: --limit @/home/centos/node-config-playbook.retry
null_resource.node_config (remote-exec): PLAY RECAP *********************************************************************
null_resource.node_config (remote-exec): infra1.openshift.local     : ok=0    changed=0    unreachable=1    failed=0
null_resource.node_config (remote-exec): infra2.openshift.local     : ok=0    changed=0    unreachable=1    failed=0
null_resource.node_config (remote-exec): master1.openshift.local    : ok=0    changed=0    unreachable=1    failed=0
null_resource.node_config (remote-exec): node-databas1.openshift.local : ok=0    changed=0    unreachable=1    failed=0
null_resource.node_config (remote-exec): node-verksamhet1.openshift.local : ok=0    changed=0    unreachable=1    failed=0
```

## Reason

This error occurs when some resources that are created by terraform are not correctly destroyed and remains in a dangling state. These resources cannot be seen through the azure portal hence, are difficult to track.

Following resources were not deleted even when `terraform destroy` was successfully executed

```
  - null_resource.bastion_config
  - null_resource.bastion_repos
  - null_resource.main
  - null_resource.node_config
  - null_resource.public_certificate
  - null_resource.template_inventory
  - module.node_bastion.module.public_ip.random_string.node
  - module.node_databas.module.public_ip.random_string.node
  - module.node_infra.module.public_ip.random_string.node
  - module.node_master.module.public_ip.random_string.node
  - module.node_verksamhet.module.public_ip.random_string.node
```
Therefore outputs error on creating again

## Precautions

Be careful when destroying/creating cluster

1. Use the same directory to create/destroy cluster (preserve tfstate files )
2. When destroying make sure all resources in the resource group for azure are deleted. (To be sure wait 5-10 mins after terraform destroy command returns)

## Solution

Run `terraform destroy -var-file=<var-file>` to make sure there are no dangling resources left.
Always run destroy form the same directory from which create was called. (to ensure same tfstate files are used)