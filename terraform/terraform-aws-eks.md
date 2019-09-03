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



