When you can't find the address allocated to a pod or a service, it's most probably allocated to a node. unfortunately this is not visible with oc commands, but you can use an ansible oneliner to find the node carrying it on the tun0 interface:

```
[ansible@vlpocpma01 ~]$ ansible -m setup -i /etc/ansible/hosts nodes -a "filter=facter_ipaddress_tun0" | grep -A2 -B2 10.234.2.1
vlpocpie01.mgmt.hh.atg.se | SUCCESS => {
    "ansible_facts": {
        "facter_ipaddress_tun0": "10.234.2.1"
    },
    "changed": false
```
