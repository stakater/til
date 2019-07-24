# Openshift sdn pods not starting

On certain AMIs, openshift sdn pods may not start causing the nodes to not become ready. This issue is caused when the network interface is not allowed to be managed by network manager. You can confirm this by reading the file `/etc/sysconfig/network-scripts/ifcfg-eth0` and make sure that `NM_CONTROLLED` is set to `yes`. To automate this, you can add the following task to your standard ansible node config.
Make sure you enable this before starting network manager.

```bash
- name: Allow network to be controlled by Network Manager
  lineinfile:
    dest: /etc/sysconfig/network-scripts/ifcfg-eth0
    regexp: '^NM_CONTROLLED=no$'
    line: 'NM_CONTROLLED=yes'
    backrefs: yes
```
