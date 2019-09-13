# Create VPC using terraform

## Overview

To create VPC using terraform follow the guidelines given below:

## Guidelines

* Clone this [repository](https://github.com/btower-labz/terraform-aws-btlabz-vpc-ha-3x)

* Change the following files:

    * az.tf
    * var-az.tf
    * version.tf

   based on your requirements. Mostly region and avalibility zones needs to be changed.

* Once configuration are changes, run following commands:

```bash

$ terraform init

$ terraform validate

$ terraform plan -out planOutput

$ terraform apply "planOutput"
```