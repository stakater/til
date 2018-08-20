# Promoting Applications Across Environments

Promoting applications across environments is always a challenge; so, the question is how to promote applications when running 
on container platform like kubernetes!

## Overview

Application promotion means moving an application through various runtime environments, typically with an increasing level of 
maturity. For example, an application might start out in a development environment, then be promoted to a stage environment 
for further testing, before finally being promoted into a production environment. As changes are introduced in the application,
again the changes will start in development and be promoted through stage and production.

The "application" today is more than just the source code written in Java, Perl, Python, etc. It is more now than the static 
web content, the integration scripts, or the associated configuration for the language specific runtimes for the application. 
It is more than the application specific archives consumed by those language specific runtimes.

In the context Kubernetes and Docker, additional application artifacts include:

- Docker container images with their rich set of metadata and associated tooling.
- Environment variables that are injected into containers for application use.
- API objects (also known as resource definitions; i.e. deployments, services, ingresses, etc.)

## References

- https://docs.openshift.com/online/dev_guide/application_lifecycle/promoting_applications.html#dev-guide-promoting-application-de
- https://docs.openshift.com/online/dev_guide/managing_images.html#dev-guide-managing-images
- https://github.com/openshift/origin/blob/master/docs/proposals/image-promotion.md
