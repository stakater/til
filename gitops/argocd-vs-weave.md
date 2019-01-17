# ArgoCD vs Weave

copied from slack

**By jessesuen**

I would say the biggest differences are:

Argo CD:
* operates at finer grained level -- syncs happen an application level instead of a repo level
* flexible in templating (kustomize, helm, ksonnet, jsonnet, yaml, replicated ship)
* provides a UI (flux’s is part of Weave Cloud)
* API/CLI is intended to be invoked from pipelines (e.g. `argocd sync APPNAME`, `argocd wait APPNAME`)
* performs health assessments on app resources
* designed for multi-tenant access/control (SSO, OIDC bindings, RBAC, projects)
* supports multiple clusters and git repos

Some of the things Flux can do that Argo CD can’t:
* monitors a docker repo for new images and can deploy from the repo
* can write back live state back to the git repo
* probably more..

**By monadic**

Flux and Argo were started on different design principles (Flux 2.5 years ago). Some things that were important to us building Flux:

Narrow scope, so it easily accommodates other tools - think of it as GitOps operator in the cluster: it makes sure that the cluster state matches the one in git and it automates deployments. This means Flux fills this one use-case - you can pick and choose whatever other tools/infrastructure you want - there's no mandate from Flux
→ one flux per cluster, easy to tell flux to track one sets of releases
For example like ArgoCD, it should be possible to use Flux with workflow tools like Argo, GH Actions (we blogged about this), and so on.
Security + compliance: https://www.weave.works/blog/gitops-compliance-and-secure-cicd
Integrations: helm (built-in), Istio, OpenFaaS, Flagger (automated canary deployments), fluxcloud (slack alerts), flux in kubeflow, etc.
Powerful fluxctl CLI

And a footnote to some of the points of Jesse earlier:
Flux has Helm integration (including image updates), though not kustomize or ksonnet
the flux API gives you the progress of deployments
you can call fluxctl from CI if you want to

overall I see argocd and flux covering more and more of the same kinds of features, so it would be great to align more if that is possible
