# Command stuck in Jenkins and exits with error

`sh` command in jenkins script gets stuck and exits with the following error:
```
process apparently never started in /home/jenkins/agent/workspace/_<JENKINS_SLAVE_NAME>@tmp/durable-a0122211
(running Jenkins temporarily with -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.LAUNCH_DIAGNOSTICS=true might make the problem clearer)
```


# Solution
- In the new kubernetes plugin in Jenkins, the default jnlp container that is created has working directory `/home/jenkins/agent`. change it to `/home/jenkins` form the Global Settings in Manage Jenkins

- Change the `builderImage`. some old pipeline-tool versions show this error