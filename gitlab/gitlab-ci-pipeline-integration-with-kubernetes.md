# Gitlab Kubernetes Integration

## Overview.

This document provides guideline on how to access kubenetes cluster from gitlab pipeline runner. The reason to use gitlab is to automate the Stakater
stacks deployment on Kubernetes. Stacks gloabl and release are used by other stack for deployment, but currently there is no way to deploy global and release
stack automatically. Therefore gitlab will be used to deploy these stacks.

## Configuration 

* Create a private project on gitlab.

* Gitlab pipeline by default uses shared runners. Shared Runners on GitLab.com run in autoscale mode and are powered by Google Cloud Platform. Autoscaling means 
  reduced wait times to spin up builds, and isolated VMs for each project, thus maximizing security. They're free to use for public open source projects and 
  limited to `2000 CI minutes per month per group for private projects`. Read about all GitLab.com [plans](https://about.gitlab.com/pricing).

* Gitlab pipeline default timeout is `1 hour`, which is not efficient because if pipeline process stucks at some step then it will consume a lot of minutes and
  we have limited number of minutes.   

* Gitlab runner is a process that run jobs, details can be found on this [link](https://docs.gitlab.com/runner/).

* There are two ways to configure gitlab with kubernetes:
  
  - `Method 1: Using Gitlab UI`: This method requires manual steps(which can be automated but it will require a lot of effort). 
                                 Guidelines can be found on this [link](https://docs.gitlab.com/ee/user/project/clusters/index.html#adding-an-existing-kubernetes-cluster). I haven't explored this method in details.

  - `Mehtod 2: Using Gitlab CI/CD Environment Variables`: In this method cluster's configurations(kube config) will be stored as a Gitlab CI/CD environment variables, which can be accessed by runner during pipeline execution. 
                                                          Details about env variables can be found on this [link](https://docs.gitlab.com/ee/ci/variables/).


