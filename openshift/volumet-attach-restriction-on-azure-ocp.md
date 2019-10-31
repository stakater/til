# Volume Attach Restriction on Azure Managed OCP

VM instances created on azure by default has a restriction to attach a limited number of block storage volumes (LUN). VM type and No. of volumes that can be attached can be seen [here](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-general)

When using K8s/Openshift we create many volumes/PVCs that need to get attached to hosts but it cannot be attached because of the restriction.

## Solution

Create a storage class of type `azure-file` instead of `azure-disk` and use it instead of block storage.

To use `azure-file` follow this [tutorial](https://docs.microsoft.com/en-us/azure/aks/azure-files-dynamic-pv)
