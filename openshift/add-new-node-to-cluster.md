# How to add new node to an on-prem OpenShift cluster?

Here are the high level steps to add new nodes to an OpenShift on-prem cluster:

**Step 1:** From Statellite provision a node by making an API call

And in the end of kickstart profile we install puppet agent

The puppet agent then reaches the puppet master to download and configures itself; the puppet master in this case Satellite as well

The puppet does basic things like configures users; add keys; NTP; etc. 

**Step 2:** Run pre-flight readiness checks with ansible


**Step 4:**


