# Gitlab Kubernetes Integration

## Overview.

This document provides guideline on how to access kubenetes cluster from gitlab pipeline runner. The reason to use gitlab is to automate the Stakater
stacks deployment on Kubernetes. Stacks global and release are used by other stack for deployment, but currently there is no way to deploy global and release
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

* A debian image containing kubectl installed will be used.

* Once everything is configured now we can access kubernetes cluster from the gitlab runner.

* One issue that might appear is how to store the kube config in CI/CD environment variables. Steps to store env vars are given below:

    1- Convert the kube config to `base64` encoded string.
    
    ```bash
    $ cat config | base64 > config.txt
    ```
    
    2- We can use this script python script to remove the next line char from the config file.
    
    ```bash
    $ base64 config.txt -w0
    ```
    
    3- If next line chart is not removed then the config cannot be decoded.

    4- Config can be decoded by using this command:
    
    ```bash
    <config> | base64 -d 
    ```

* Gitlab `.gitlab-ci.yml` file uses the manifest given below for configuration:
  ```bash
  export BRANCH="azure-capability"
  export INSTALL_PVC="false"
  export NAMESPACE="monitoring"
  export PROVIDER="aws"
  export REPO_URL="github.com/stakater/StakaterKubeHelmMonitoring.git"
  export TARGET="install-dry-run"
  ```

*  Gitlab pipeline manifest is given below:

  ```yaml
  image:
    name: stakater/gitlab:0.0.3

  before_script:

    - if [ $STACK == "global" ]; then \
    -     echo "Gloabl Stack"; \
    -     source ./global.sh; \
    
    - elif [ $STACK == "release" ]; then \ 
    -     echo "Release Stack"; \ 
    -     source ./release.sh; \
    
    - elif [ $STACK == "logging" ]; then \
    -     echo "Logging Stack"; \ 
    -     source ./logging.sh; \
    
    - elif [ $STACK == "monitoring" ]; then \
    -     echo "Monitoring Stack"; \ 
    -     source ./monitoring.sh; \
    
    - elif [ $STACK == "tracing" ]; then \
    -     echo "Tracing Stack"; \ 
    -     source ./tracing.sh; \
    
    - else \
    -     echo "Invalid stack name provided"
    -     exit 1 
    - fi

    - echo "Displaying non-sensitive config parameters on pipeline logs"
    - echo $BRANCH
    - echo $INSTALL_PVC
    - echo $NAMESPACE
    - echo $PROVIDER
    - echo $REPO_URL
    - echo $TARGET

    - echo "configuration kubernetes access in pipeline"
    - mkdir ~/.kube/
    - echo $KUBE_CONFIG | base64 -d > config
    - mv config ~/.kube/

    - echo "Extracting repository name from the URL"
    - BASE_NAME=$(basename $REPO_URL)
    - REPO_NAME=${BASE_NAME%.*}

  stages:
    - deploy

  deploy:
    stage: deploy
    script:
      
      - echo "Cloning repository and redirecting the output to black hole because we don't want GITHUB_TOKEN to be visible on pipeline logs"
      - git clone https://$GITHUB_TOKEN@$REPO_URL > /dev/null

      - echo "Moving inside cloned repository directory"
      - cd $REPO_NAME

      - echo "Checkout to the deployment branch"
      - git checkout $BRANCH

      - echo "Deploying stack on kubernetes cluster"

      #- if [ -z "$NAMESPACE" ]; then       make $TARGET PROVIDER_NAME=$PROVIDER INSTALL_PVC=$INSTALL_PVC; else       make $TARGET NAMESPACE=$NAMESPACE PROVIDER_NAME=$PROVIDER INSTALL_PVC=$INSTALL_PVC; fi
      - if [ -z "$NAMESPACE" ]; then \
      -       make $TARGET PROVIDER_NAME=$PROVIDER INSTALL_PVC=$INSTALL_PVC; \
      - else \
      -       make $TARGET NAMESPACE=$NAMESPACE PROVIDER_NAME=$PROVIDER INSTALL_PVC=$INSTALL_PVC; \      
      - fi
  ```

* Pipeline required following CI/CD environment variables:

| Gitlab CI/CD Environment Variables | Description |
|---|---|
| KUBE_CONFIG_AWS  | Kubernetes configuration file. It must be provided as base64 encoded string. Guideline on how to encode and decode kube config can be found on this [link](https://github.com/stakater/til/blob/master/gitlab/gitlab-ci-pipeline-integration-with-kubernetes.md#using-gitlab-cicd-environment-variables) |
| NAMESPACE  | Deployment namespace. |
| PROVIDER  | Cloud provider's name. Currently, we support AWS and Azure. Its value can be provided as `aws` for AWS and `azure` for Azure. |
| REPO_URL  | URL of the stack repository that need to be deployed. From each repository URL only the part after this section `https://` is required. Like from `https://github.com/stakater/til.git` URL we require only the `github.com/stakater/til.git` part |
| GITHUB_TOKEN  | Github Personal Access token. It can be generated by following this [guideline](https://github.com/stakater/til/blob/master/gitlab/gitlab-integration-with-github.md). |
| BRANCH  | Branch name that will be deployed. |
| TARGET  | Makefile target's name that will be executed. |
| INSTALL_PVC | PVC installation is required or not. Its values are `true` and `false` |


If anything needs to change regarding these CI/CD env variables. Just change them at these locations:
* .gitlab-ci.yml.
* Gitlab CI/CD environment variables.

## Refrences

* https://about.gitlab.com/2017/09/21/how-to-create-ci-cd-pipeline-with-autodeploy-to-kubernetes-using-gitlab-and-helm/