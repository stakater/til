# Using EBS with ReadWriteMany policy

You can't use ReadWriteMany policy with EBS, as stated here: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes

A workaround is to use EFS with dynamic provisioning using [EFS Provisioner](https://github.com/kubernetes-incubator/external-storage/tree/master/aws/efs).