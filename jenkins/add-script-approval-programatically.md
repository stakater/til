# Add script approval to jenkins programatically

We can run the following in the script console of jenkins

```groovy
def signature = 'new groovy.json.JsonSlurperClassic'
org.jenkinsci.plugins.scriptsecurity.scripts.ScriptApproval.get().approveSignature(signature)
```
