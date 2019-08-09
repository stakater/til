# Postgresql Volume Permissions issue

Postgres chart uses user 1001 (user: postgres) for CRUD operations. But IBM's file storage volumes only allows root user to perform operations on its file system. Therefore instead of using `ibmc-file-silver` type storage. we will use `ibmc-block-silver` type storage.

`ibmc-block` type provisioner is a plugin that needs to be deployed separately (by default only `ibmc-file`). The block storage plugin can be deployed using following (right now gets deployed in control stack `pre-install` step)
```
helm repo add iks-charts https://icr.io/helm/iks-charts || true
helm repo update || true
helm install iks-charts/ibmcloud-block-storage-plugin || true
```