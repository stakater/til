# Logging Operator creates too many PVC on upgrade

Logging operator creates too many PVCs and sometimes bind wrong volumes to the deployments. THis is a known bug and is fixed in OCP 4.6.1

## Temporary fix
https://access.redhat.com/solutions/5323141


## Permanent Fix
Upgrade to OCP 4.6.1