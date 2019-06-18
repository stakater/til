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

  - `Mehtod 2: Using Gitlab CI/CD Environment Variables`: In this method cluster's configurations(kube config) will be stored as a Gitlab CI/CD environment variables, which can be accessed by runner during pipeline execution. Details about env variables can be found on this [link](https://docs.gitlab.com/ee/ci/variables/). Details can be found below.



### Using Gitlab CI/CD Environment Variables

The section provides guidelines on how to access kubernetes cluster using Gitlab CI/CD environment variables.

* New CI/CD env variables can be created by going to `Setting -> CI/CD -> Variables`. Add the variable (named as `KUBE_CONFIG` and it will store kubernetes cluster config) and save it. <add security aspects here>

* Now create a `.gitlab-ci.yaml` in the project. It will contains the steps that will be executed in pipeline.

* In pipeline first we will extract the value of `KUBE_CONFIG` and store it as kubernetes config.

* A debain image containing kubectl installed will be used.

* Once everything is configured now we can access kubernetes cluster from the gitlab runner.

* To store different cloud providers kube config we can name the kubeconfig variables in this format: `KUBE_CONFIG_AZURE`, `KUBE_CONFIG_AWS`. One issue that might appear is how to store the kube config in CI/CD environment variables. Steps are given below:

    1- Convert the kube config to `base64` encoded string.
    ```bash
    $ cat config | base64 > data.txt
    ```
    2- We can use this script python script to remove the next line char from the config lines.

    ```python
    f = open("guru99.txt", "r")
    config = ""
    for line in f:
      config += line.split("\n")
    print(config)
    ```
    3- If next line chart is not removed then the config cannot be decoded.
    4- Config can be decoded by using this command:
    ```bash
    <config> | base64 -d 
    ```

*  Gitlab pipeline manifest is given below:

```yaml
image:
  name: aliartiza75/kubectl:0.0.2

before_script:
  - mkdir ~/.kube/
  - echo $KUBE_CONFIG_AWS | base64 -d > config
  - mv config ~/.kube/

stages:
  - deploy

deploy:
  stage: deploy
  script:
    - kubectl get namespaces
    - make install NAMESPACE=logging
```


## Refrences

* https://about.gitlab.com/2017/09/21/how-to-create-ci-cd-pipeline-with-autodeploy-to-kubernetes-using-gitlab-and-helm/