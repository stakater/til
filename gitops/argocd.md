# ArgoCD

## Benefits
1. Nice UI to visualize the resources that are deployed, and which ones are out of sync, if any.
2. Supports manifests, helm charts, and ksonnet applications.
3. Supports SSO
4. Supports git webhooks
5. Tracking strategies for branch, tag and commit. i.e. it can watch any of these and then sync whenever there is a change

## Drawbacks
1. An application needs to be registered in Argo-CD. So for the case of one git repository with multiple applications, each application will have to be registered individually in ArgoCD. This will can be done via command line or API.
2. Uses a CRD for application. So state is not persisted 
3. Currently helm hooks are ignored. However they are planning to map them to ArgoCD's own pre/post-sync hooks

## Compared with Weave Flux
* Both projects are under active development, however ArgoCD is more stable with more features than Flux, so it may stay ahead in these terms for now.
* Both require a CRD to be created for each project
* For secrets, Bitnami sealed secrets will have to be used in either case
