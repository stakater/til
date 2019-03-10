# Business problem and business value

## Business Problem

Businesses today want to deliver new features and updates to their products for their internal users as well as
external stakeholders quickly and with high quality. Every industry today is seeing a transformation, which is
predominantly driven by advances in technology. In order to stay competitive and relevant in their respective
industry and marketplace, every business needs to take advantage of new technologies quickly and adopt
them to their products and solutions. Today, much of the technology advancement and innovation is driven
through a combination of software and hardware. More importantly, emerging technologies such as artificial
intelligence (AI) and machine learning (ML) are fuelled by rapid advancements in software. In addition, many
of the IT infrastructure and data center advancements are driven through software defined technologies such
as software defined storage (SDS) and software defined networking (SDN). Hence software is a key driver in
pushing forward various technologies in all industries. 

Container technology has picked up momentum in the software development area and enabled developers to
take advantage of several benefits from packaging their applications as containers:

- Containers are light-weight application run-time environments compared to virtual machines and are therefore less resource intensive and highly efficient.
- Containers enable developers to package their applications as well as all the library dependencies to properly run them so that a container image provides a completely self-sufficient environment to execute the application code. This also means that multiple application instances requiring different versions of the same libraries can be packaged into different containers and run side-by-side on the same operating system instance without any interference.
- Containers are portable across different platforms (as long as the underlying operating systems are compatible). Docker is a well-known open source project that provides the run-time abstraction and facilities to build and run containerized applications in a portable fashion.
- Containers are now the de facto standard of operation for some of the well-known public cloud environments including Google, Amazon AWS, and Microsoft Azure. Hence, applications packaged as containers can be executed on-prem or on public cloud without any modifications (other than additional steps to security the software for use on the public cloud).
- There are many open source tools available now to help developers easily create, test, and deploy containerized applications. In addition, all the well-known protocols for security, authentication,/authorization, storage, etc., can be applied to containerized workloads without any modifications to applications. In other words, you can take a legacy application written in a language such as Java and package it, as is, into a container image and run it.
- Containers are now the way to implement a continuous integration/continuous delivery (CI/CD) development pipeline and the DevOps paradigm of combining software development and infrastructure operations. 

## Business Values

Software development life cycle (SDLC) practices have evolved to achieve high velocity and efficiency of
development. Organizations today implement Agile/Scrum as the primary methodology to create cohesive
development teams that work close to their customers, gather incremental product requirements, and deliver
new features in short development cycles.

### DevOps Overview

DevOps is evolving as a standard practice in many organizations to bring together software development and
IT operations teams for the goal of eliminating process bottlenecks in development, quality assurance (QA),
and delivery cycles, and servicing their end customers efficiently. Implementing a proper DevOps process
requires careful planning and an assessment of the end-to-end pipeline from development to QA to delivery.
Automation is a key aspect of DevOps. Traditional software development processes were handled mostly
manually. When developers commit code to the source code repository, the test engineer or a build engineer
would then checkout the code and build the project, resolving any conflicts. After the QA iterations, the release
engineer would be responsible to take the release branch code and build the final shippable product. Along
this pipeline, many of the steps were handled manually by people, which introduced the delays in the release
cycles. Agile development now takes advantage of new automation tools that remove the manual steps.
Another core aspect of DevOps is providing the necessary freedom and resources to develop and test code
without having to rely on IT operations teams to re-provision or re-configure hardware every time. With the
advances in technologies including virtualization, containers, Cloud multi-tenancy, self-service, and so on, it is
now possible to detach applications and end-users from physical hardware and provide the necessary tools
for them to create the right virtual environment to run their applications without directly modifying the physical
hardware or interfering with other users’ applications. With cloud self-service, users can request and provision
the hardware to meet their application specific requirements. Cloud administrators create the proper policy
and authorization workflows such that the provisioning process does not require manual steps. DevOps
essentially combines the role of the software engineer with that of the IT operator so that the end-to-end
software pipeline can be implemented with automation.

### Monolithic Vs Micro-services Architecture

Software architecture over the last two to three decades has evolved from a monolithic application that
essentially delivered all feature functions in a single package to service-oriented architectures where the
application is divided up into multiple tiers with each tier providing programming interfaces (APIs) for its clients
to access the features via service calls. Software principles such as modularity, coupling, code reuse, etc.,
have remained the core principles that people still use, however new programming languages, runtime
facilities, mobile versus cloud native techniques, etc., have evolved in the recent years to shift software from
the traditional architectures to more of a micro-service architecture.
Micro-services are software applications that are organized around smaller subsets of functionality of the
overall application such that they are much more manageable than a bulky piece of software developed by
10s of developers and coordinated in a complex dev/test process. Micro-services are software modules that
run as services with open APIs. They use open protocols, e.g. HTTP, and expose REST based APIs so that
the services can run anywhere - on-premises or on the public cloud, and still be locatable via well-defined 
service end-points. Due to this design, micro-services provide a loosely-coupled architecture that can be
maintained by smaller development teams and can be independently updated.
Containers provide a natural mechanism to implement micro-services because they allow you to package the
application code and all its runtime library dependencies into a single image, which is portable across various
platforms. In addition, container orchestration platforms such as Kubernetes provide the mechanisms for
service location, routing, service replication, etc., which helps micro-services development and runtime.
Developers do not need to explicitly write additional code for these types of services because the platform
provides these facilities. 

### Continuous Integration/Continuous Delivery (CI/CD)

As discussed in the previous section, successful DevOps practice requires a good amount of automation of
the development, test, QA, and delivery pipeline. This is where the CI/CD comes into play.
Continuous integration is the process by which new code development through build, unit testing, QA, and
delivery is automated end-to-end using build tools and process workflows. CI enables rapid integration of
code being developed by multiple engineers concurrently and committed into a source code repository. CI
enables rapid build and test of code so that software bugs and quality issues are identified quickly. Once the
code passes the test plan and QA it can then be pushed to the release branches for release integration.
Continuous Delivery (CD) enables automation around delivering code to production systems after performing
the necessary functional, quality, security, and performance tests. Continuous delivery enables bringing new
features in the software to the end users faster without going through the manual release test and promotion
steps.
More information on CI/CD with OpenShift is available in the following online book:
assets.openshift.com/hubfs/pdfs/DevOps_with_OpenShift.pdf

As described previously, source code from multiple concurrent developers is integrated, tested, and deployed
to production through automation tools. The OpenShift Container Platform provides the mechanisms to
implement the CI/CD pipelines with tools such as CloudBees Jenkins. See the following post on how to do
this: blog.openshift.com/cicd-with-openshift/.

