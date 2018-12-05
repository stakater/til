# Delete multiple pods with Same labels

You can delete multiple pods with same labels through a single command `kubectl delete pods -l labelName=labelValue -n namespace`

The above command is helpful to restart all pods of a deployment, stateful set or others.