# Volume Attach Restriction on Azure Managed OCP

VM instances created on azure by default has a restriction to attach a limited number of block storage volumes (LUN). VM type and No. of volumes that can be attached can be seen [here](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-general)

When using K8s/Openshift we create many volumes/PVCs that need to get attached to hosts but it cannot be attached because of the restriction.

## Solution

Resize the VMs to a different type. We have resized to DS2_v2 for Infra nodes. which gives
- 2 vCPUs
- 7 GB RAM
- 8 Data Disks
- 6400 IOPS
- 14 GB Temp. Storage 

## How to Resize.

- Select the node you want to resize from resource group
- On the left panel select size, It will display different sizes that are available for resizing
- Select the type you want to resize your node to
- Click `Resize` to resize. (The VM will be restarted when resizing.)