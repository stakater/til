# ArgoCD 

## Benefits
1. Nice UI to visualize the resources that are deployed, and which ones are out of sync, if any.
2. Supports manifests, helm charts, and ksonnet applications.
3. Supports SSO
4. Supports git webhooks
5. Tracking strategies for branch, tag and commit. i.e. it can watch any of these and then sync whenever there is a change

## Drawbacks
1. Autosync not yet implemented, though is on the roadmap. An application sync will have to be triggered via REST Api or command line. And these will be triggered from the jenkins pipeline for automation.
2. An application needs to be registered in Argo-CD. So for the case of one git repository with multiple applications, each application will have to be registered individually in ArgoCD. This will can be done via command line or API.
3. Uses a CRD for application. So state is not persisted 
4. Currently helm hooks are ignored. However they are planning to map them to ArgoCD's own pre/post-sync hooks
