# Docker Image Promotion

Describe the best practices and define pattern for promoting Docker images between different stages (e.g. from development to production).

Read this openshift [proposal](https://github.com/openshift/origin/blob/master/docs/proposals/image-promotion.md)

Docker images can be promoted (and the use-cases are valid for):

- Within a single Namespace/Project
- Between multiple Namespaces/Projects
- Between multiple Clusters

The Docker image promotion can be done using the Docker image tags or Docker image labels.

Why do image promotion?

Application promotion means moving an application through various runtime environments, typically with an increasing level 
of maturity. For example, an application might start out in a development environment, then be promoted to a stage environment 
for further testing, before finally being promoted into a production environment. As changes are introduced in the 
application, again the changes will start in development and be promoted through stage and production.

## Deployment Environments

A deployment environment, in this context, describes a distinct space for an application to run during a particular stage 
of a CI/CD pipeline. Typical environments include development, test, stage, and production, for example. The boundaries 
of an environment can be defined in different ways, such as:

- Via labels and unique naming within a single project.
- Via distinct projects within a cluster.
- Via distinct clusters.

And it is conceivable that your organization leverages all three.

### Considerations

Typically, you will consider the following heuristics in how you structure the deployment environments:

- How much resource sharing the various stages of your promotion flow allow
- How much isolation the various stages of your promotion flow require
- How centrally located (or geographically dispersed) the various stages of your promotion flow are

After you understand the components of your application that need to be moved between environments when promoting it and the 
steps required to move the components, you can start to orchestrate and automate the workflow. 

Of the components making up an application, the image is the primary artifact of note. Taking that premise and extending it 
to application promotion, the core, fundamental application promotion pattern is image promotion, where the unit of work is 
the image. The vast majority of application promotion scenarios entails management and propagation of the image through the 
promotion pipeline.

Simpler scenarios solely deal with managing and propagating the image through the pipeline. As the promotion scenarios 
broaden in scope, the other application artifacts, most notably the API objects, are included in the inventory of items 
managed and propagated through the pipeline.

This is a [MUST READ](https://docs.openshift.com/online/dev_guide/application_lifecycle/promoting_applications.html)

