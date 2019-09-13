# How to use ECR with pipeline library

In order to use ECR pipeline library 3 things are needed:
1. Run the following command to get credentials from ECR registry. (supported in stakater-pipeline-library from version 2.16.5)
```
$(aws ecr get-login --no-include-email --region `region` )
```
2. A binary `amazon-ecr-credential-helper` (added in pipeline-tools v2.0.12)
3. Add the following in dockerconfigjson 
```
{ "credsStore": "ecr-login" }
```
4. In ECR give Permission to PUSH/PULL form ECR registry to role: `nodes.stackator.com`

## How it works

Jenkins slaves have role: `nodes.stackator.com`. They can access ECR by the get-login command in step 1 via `amazon-ecr-credential-helper` (done by stakater-pipeline-library) to get a token which is used to push/pull from ECR.