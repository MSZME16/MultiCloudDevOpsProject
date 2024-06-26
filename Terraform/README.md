# Terraform Infrastructure for MultiCloudDevOpsProject

## Overview

This Terraform configuration provisions the necessary AWS infrastructure to support a Jenkins server used for a DevOps pipeline. The infrastructure includes VPC, subnets, EC2 instances, and CloudWatch monitoring.

## Directory Structure

```
terraform/
├── ec2/
│   ├── main.tf
│   ├── outputs.tf
│   └──variables.tf
├── vpc/
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── subnet/
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── cloudwatch/
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── main.tf
├── remote_backend.tf
├── variables.tf
├── terraform.tfvars
└── README.md
```

## Modules

### VPC Module

#### Purpose
The VPC module creates a Virtual Private Cloud (VPC) in AWS to host your resources.

#### Files
- **main.tf**: Defines the VPC resource.
- **outputs.tf**: Outputs the VPC ID.
- **variables.tf**: Defines the input variables for the VPC.

```hcl
module "vpc" {
  source             = "./vpc"
  vpc_cidr           = var.vpc_cidr
}

```
![alt text](../screenshots/vpc.png)

### Subnet Module

#### Purpose
The subnet module creates public and private subnets within the VPC.

#### Files
- **main.tf**: Defines the subnet resources.
- **outputs.tf**: Outputs the subnet IDs.
- **variables.tf**: Defines the input variables for the subnets.

```hcl
module "subnet" {
  source             = "./subnet"
  vpc_id             = module.vpc.vpc_id
  subnet             = var.subnet
  igw_id             = module.vpc.igw_id
}
```
![alt text](subnet.png)

### EC2 Module

#### Purpose
The EC2 module provisions EC2 instances to host Jenkins.

#### Files
- **main.tf**: Defines the EC2 instance resources.
- **outputs.tf**: Outputs the instance IDs and public IP addresses.
- **variables.tf**: Defines the input variables for the EC2 instances.

```hcl
module "ec2" {
  source             = "./ec2"
  sg_vpc_id          = module.vpc.vpc_id
  public_key_path    = var.public_key_path
  ec2_ami_id	       = var.ec2_ami_id
  ec2_type  	       = var.ec2_type
  ec2_subnet_id      = module.subnet.subnet_id
}

```
![alt text](../screenshots/ec2.png)

### CloudWatch Module

#### Purpose
The CloudWatch module sets up monitoring for the EC2 instances.

#### Files
- **main.tf**: Defines the CloudWatch resources.
- **outputs.tf**: Outputs the CloudWatch dashboard and alarms.
- **variables.tf**: Defines the input variables for CloudWatch.

```hcl
module "cloudwatch" {
  source              = "./cloudwatch"
  ec2_id	            = module.ec2.ec2_id
  cloudwatch_region   = var.region
  sns_email	          = var.sns_email
}
```

![alt text](../screenshots/cloud_watch1.png)

![alt text](../screenshots/cloud_watch_dashboard.png)

## Usage Instructions

### Prerequisites

- Install [Terraform](https://www.terraform.io/downloads.html)
- AWS CLI configured with appropriate IAM permissions
- SSH key pair created in AWS (for EC2 access)

### Steps

1. **Navigate to the Terraform Directory**:
```bash
   cd Terraform
```
2. **Initialize Terraform**:
```bash
  terraform init
```
![alt text](../screenshots/terraform1.png)

3. **Provide AWS Credentials**:

Ensure that AWS credentials are configured either through environment variables or AWS shared credentials file.

4. **Apply Infrastructure Changes**:

```bash
terraform apply -auto-approve
```
![alt text](../screenshots/terraform.png)


### S3 and DynamoDB

![alt text](../screenshots/s3.png)

![alt text](../screenshots/DynamoDB.png)

5. **Destroy Infrastrucure After Finish**

![alt text](../screenshots/terraform_destroy.png)

## Variables

### VPC Module

- `vpc_name` (string): The name of the VPC.
- `cidr_block` (string): The CIDR block for the VPC.

### Subnet Module

- `vpc_id` (string): The ID of the VPC.
- `public_subnet_cidr` (list): CIDR blocks for public subnets.
- `private_subnet_cidr` (list): CIDR blocks for private subnets.

### EC2 Module

- `instance_type` (string): The type of EC2 instance (e.g., `t2.medium`).
- `ami_id` (string): The ID of the AMI to use for the EC2 instance.
- `key_name` (string): The name of the SSH key pair.
- `vpc_security_group_ids` (list): The security group IDs to associate with the instance.
- `subnet_id` (string): The ID of the subnet to launch the instance in.

### CloudWatch Module

- `ec2_instance_ids` (list): The IDs of the EC2 instances to monitor.

## Outputs

### VPC Module

- `vpc_id`: The ID of the created VPC.

### Subnet Module

- `public_subnet_id`: The ID of the created public subnet.
- `private_subnet_id`: The ID of the created private subnet.

### EC2 Module

- `instance_ids`: The IDs of the created EC2 instances.
- `public_ips`: The public IP addresses of the created EC2 instances.

### CloudWatch Module

- `cloudwatch_dashboard_url`: The URL of the CloudWatch dashboard.
- `cloudwatch_alarms`: The created CloudWatch alarms.
