# How to run a specific playbook?

- Login into master node
- `sudo su` to become `root`
- `su ansible` to become `ansible` user
- then `cd /home/ansible/git/ocp-ansible/`
- git branch verify its master
- git pull
- then to run a playboo while sitting in `ocp-ansible`
`ansible-playbook -i devtest --vault-password-file=~/ansible-password.txt -e openshift_disable_check=disk_availability /usr/share/ansible/openshift-ansible/playbooks/byo/openshift-cluster/openshift-logging.yml`
