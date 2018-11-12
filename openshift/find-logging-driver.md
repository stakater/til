How to find logging driver on each node of an OpenShift cluster?

ssh into master node and then become `root` with `sudo su` and then shift to `ansible` with `su ansible`

Just need a ONE liner using ansible to fetch this required info:

```
[ansible@vlpocpma01 rasheed.amir]$ ansible -i /etc/ansible/hosts nodes -a "docker info" | grep "Logging Driver:"
Logging Driver: json-file
Logging Driver: journald
Logging Driver: journald
Logging Driver: json-file
Logging Driver: journald
Logging Driver: json-file
Logging Driver: json-file
Logging Driver: json-file
Logging Driver: json-file
Logging Driver: json-file
Logging Driver: json-file
Logging Driver: json-file
Logging Driver: journald
Logging Driver: json-file
Logging Driver: json-file
Logging Driver: journald
Logging Driver: journald
Logging Driver: journald
Logging Driver: journald
Logging Driver: journald
```
