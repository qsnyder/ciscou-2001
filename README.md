# CISCOU-2001

This repository will serve as the artifactory for the CISCOU-2001 presentation at CLUS 2023 in Las Vegas.  The purpose of this reporistory will be to serve as a holding place for the demo code such that any user can instantiate their own EC2 instance with Terraform and explore a large-scale k8s cluster using KIND.

## Prerequisites
This lab assumes the use of an AWS account.  By default, the Terraform configurations will stand up an instance in `us-east-1` of a `c5a.8xlarge` instance, with 32vCPUs, 64GB of RAM, and a mapped drive of 75GB of GP2 storage.  This will come at a cost of around $1.50/hr USD.  This will also require an AWS account with a valid billing method on file.  If sufficient compute power is found outside of AWS, the rest of the files can be used, assuming Docker, KIND, and other prerequisites are installed prior to use.

An AWS SSH keypair, as well as an IAM user access and secret key will need to be generated using the AWS control panel.  The keypair will need to be saved to the local machine in an accessible location and saved using "400" permissions (only read by the user).

## Usage

Within the `terraform/terraform.tfvars` file, there are several pieces of information that will need to be filled in, including the AWS access and secret keys for the IAM user, as well as the name of the SSH keypair and its location on the local machine that will be running the Terraform configuration.  Once this information is filled in appropriately, the Terraform configuration can be run from the `terraform` folder.

```bash
terraform init
terraform plan -out tf.plan
terraform apply "tf.plan"
```

This will instantiate an EC2 instance within AWS, as well as print out the FQDN and the 
