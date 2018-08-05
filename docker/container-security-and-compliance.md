# Container Security & Complaince

## What’s Inside Your Container Images?

With Docker and containers it’s never been easier to deploy and run any application.  Developers now have access to thousands of applications ready to run right “off the shelf” and the ability to quickly build and publish their own images.

In addition to the application, the container image may contain hundreds of packages and thousands of files including binaries, shared libraries, configuration files, and 3rd party modules. Any one of these components may contain a security vulnerability, an outdated software module, a misconfigured configuration file or simply fail to comply with your operational or security best practices.

## anchore vs clair

At the same time we were working on Anchore CoreOS released the Clair project which provided an open source vulnerability scanner. We are big fans of the work CoreOS has done in the container community so we looked into that project but saw a number of gaps: firstly its focus was reporting on operating system CVEs (vulnerabilities). While CVE scanning is an important first step it is just the tip of the iceberg, a container security and compliance tool should be looking at policies that cover licensing, secrets, configuration, etc. The second challenge we saw was that Clair was focused more on the registry use case which given the Clair use in the CoreOS Quay registry made perfect sense. So we built a series of tools to address container scanning and compliance from the ground up. Since then we have been glad to see more open source container security solutions come to market such as Sysdig’s Falco runtime security project.

source: https://anchore.com/blog/driving-open-source-container-security-forward/#

## The tip of the IceBerg

https://anchore.com/about-anchore/#iceberg

Image scanning solutions focus on scanning the operating system image for known vulnerabilities (CVEs). While this is a critical check to perform it is just the first step. An image may contain no operating system packages with known vulnerabilities but may still be insecure, mis-configured or in some other way out of compliance.

## falco

https://github.com/draios/falco/
