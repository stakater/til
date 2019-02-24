# How to add new node to an on-prem OpenShift cluster?

Here are the high level steps to add new nodes to an OpenShift on-prem cluster:

**Step 1:** From Statellite provision a node

**Step 2:** Use puppet modules to add key

- this is mandatory for ansible user to be able to run playbooks

**Step 3:** Run pre-flight readiness checks with ansible

Step 4: 
