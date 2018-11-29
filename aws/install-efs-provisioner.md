# Install efs-provisioner

To install efs-provisioner, create/update the efs security group with following inbound rule.

```text
Type       : NFS
Protocol   : TCP
Port Range : 2049
Source     : 182.20.0.0/16 (private ip range of your nodes)
```

Then change these values in efs-provisioner public chart according to your efs configurations and apply the chart

```text
efsFileSystemId: fs-1234567
awsRegion: eu-west-1
path: /
```

## Mount EFS to node

```bash
sudo mkdir -p /mnt/efs;
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 fs-blahblah.efs.us-west-2.amazonaws.com:/ /mnt/efs;
```
