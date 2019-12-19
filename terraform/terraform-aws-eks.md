## Problem-1

### Error-1

```bash
Error: Error creating EIP: AddressLimitExceeded: The maximum number of addresses has been reached.
	status code: 400, request id: 7368d37c-d85f

  on .terraform/modules/eks.vpc/terraform-aws-modules-terraform-aws-vpc-271b6a7/main.tf line 770, in resource "aws_eip" "nat":
 770: resource "aws_eip" "nat" {

Error: Error creating EIP: AddressLimitExceeded: The maximum number of addresses has been reached.
	status code: 400, request id: 9

  on .terraform/modules/eks.vpc/terraform-aws-modules-terraform-aws-vpc-271b6a7/main.tf line 770, in resource "aws_eip" "nat":
 770: resource "aws_eip" "nat" {

Error: Error creating EIP: AddressLimitExceeded: The maximum number of addresses has been reached.
	status code: 400, request id: f5430a61-6527a

  on .terraform/modules/eks.vpc/terraform-aws-modules-terraform-aws-vpc-271b6a7/main.tf line 770, in resource "aws_eip" "nat":
 770: resource "aws_eip" "nat" {

```

### Related Links:

* https://github.com/hashicorp/terraform/issues/6018

* https://docs.aws.amazon.com/AWSEC2/latest/APIReference/errors-overview.html

* https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-limit


### Solutions

Remove the elastic ips using the dashboard because terraform somehow failed to remove the elasitc ips. By default aws supports 5 elastic ips per region. Although more elastic ips can also be requested.

## Problem-2 

### Error

eks utility in aws module 

aws-cli/1.16.204 Python/3.6.8 Linux/5.0.0-25-generic botocore/1.12.194

### Solution

Upgrade the aws-cli version


### Problem-3

Third problem is related to granting access to users because the current issue is that the machine from which the cluster is deployed. Only that machine has the access to access the cluster.


One way to grant access is by editing the `configmap/aws-auth` config map in the `kube-system` namespace. [Link](https://stackoverflow.com/questions/50791303/kubectl-error-you-must-be-logged-in-to-the-server-unauthorized-when-accessing)

What we can do is that the for the users we want to grant access, we will modify the configuration of the `aws-auth`.

Second issue is that if we run the pipeline in the gitlab runner, all the configuration related to eks cluster will be lost once the pipeline execution is completed.


third point it that the `aws cli` must be configured on the node because in kubeconfig it uses awscli for authentication.

## Problem-4

It is related to the deletion of the cluster and its resources. We can easily deploy the cluster using the `master` branch of the `terraform-aws-eks` repository. 

But when we try to delete the cluster we get this error that the subnets for the vpc doesn't exist. After looking at the logs of the gitlab pipeline I found out that the subnets for the vpc gets created successfully and their information also exists in `terraform.tfstate` file. But somehow terraform doesn't know the subnets.

In other case if we use the already existsing vpc and its subnets then the cluster is deployed and deleted successfully.

### Solution
This issue can be resolved by using and existing vpc.

## Problem-5

eks allow specific number of pod on a node although the resource memory and cpu is not utilized completely.

Number of pods per node can be found using this table.

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI


While deleting the eks cluster make sure these resources are deleted successfully:
These things will only be deployed when we are using new vpc.
```
autoscaling group
nat gateways
interfaces
eks-cluster
delete cloudwatch los
elastic ips
load balancers
EFS
```

## Problem-6
Sometime vpc subnets have tags on them that cause them the cluster nodes to not get deployed.

### Solution

Delete the tags


## Problem-7

cluster was deployed on the `us-east-2` successfully but it was not being deployed on `eu-west-1`. 

## Problem-8

AWS credentials issue was caused by the invalid aws configuration.

### Solution
This issue can be resolved by creating a new aws credentials like id and secret.

## Problem-9

Sometime Cluster worker nodes are not accessible by the master. This issue still exists, therefore i deployed the cluster in another vpc.


## Problem-10
I faced another issue that aws-iam-authenticator is not detected in $PATH. Although path existed in host system.

### Solution
I resolved this issue by passing another argument in the configurations

```tf
kubeconfig_aws_authenticator_command = "/root/bin/aws-iam-authenticator"
```
the above argument specifies the absolute path of the `aws-iam-authenticator` binary in the docker image.







