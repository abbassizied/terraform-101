# Get started with Terraform on AWS
> Learn how to create AWS infrastructure with Terraform

- Use Terraform to create and manage your infrastructure as code. In this tutorial, you will use Terraform to provision an EC2 instance on Amazon Web Services (AWS). EC2 instances are virtual machines running on AWS and a common component of many infrastructure projects. To provision your infrastructure, you will write configuration to define your provider and instance, set environment variables for your AWS credentials, initialize a new local workspace, and then apply your configuration to create your instance.

```sh
cd lab-02-learn-terraform-get-started-aws

# Format configuration
$ terraform fmt

# Initialize your workspace
$ terraform init

# Validate configuration
$ terraform validate

# Create infrastructure
$ terraform apply

# Inspect state
$ terraform state list

# Print out your workspace's entire state
$ terraform show 
```

## terraform.tf
 
```tcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.92"
    }
  }

  required_version = ">= 1.2"
}
```

## main.tf

- This environment uses Localstack to simulate an AWS environment. The provider "aws" block includes settings to allow the AWS provider to connect to Localstack.
```tcl
provider "aws" {
  region = "us-west-2"

  access_key                  = "anaccesskey"
  secret_key                  = "asecretkey"
  s3_use_path_style           = true
  skip_credentials_validation = true
  skip_metadata_api_check     = true
  skip_requesting_account_id  = true

  endpoints {
    ec2        = "http://localhost:4566"
  }
}

data "aws_ami" "linux" {
  most_recent = true

  filter {
    name = "name"
    values = ["amzn2-ami-hvm-2.0.*-x86_64-ebs"]
  }

  owners = ["137112412989"] # Amazon
}

resource "aws_instance" "app_server" {
  ami           = data.aws_ami.linux.id
  instance_type = "t2.micro"

  tags = {
    Name = "hashicorp-learn"
  }
}
```

##
 
```sh
# Format configuration
terraform fmt

# Create infrastructure
terraform init

# 
aws configure list
terraform apply

# List state
terraform state list

# Show state
terraform show
```

---

# Manage infrastructure

- Use variables and output values to parameterize your Terraform workspace and expose infrastructure data for external tools to reference.
- **Input variables** let you parametrize the behavior of your Terraform configuration.
- **Output values** allow you to access attributes from your Terraform configuration and consume their values with other automation tools or workflows.



## variables.tf
 
```tcl
variable "instance_name" {
  description = "Value of the EC2 instance's Name tag."
  type    	   = string
  default 	   = "hashicorp-learn"
}

variable "instance_type" {
  description = "The EC2 instance's type."
  type    	   = string
  default 	   = "t2.micro"
}
```

## outputs.tf
 
```tcl
output "instance_hostname" {
  description = "Private DNS name of the EC2 instance."
  value       = aws_instance.app_server.private_dns
}
```

## Plan and apply changes
 
```sh
terraform apply
```

## Modules

- **Modules** are reusable collections of resources. Like providers, you can source modules from the Terraform Registry. You can also create and publish your own modules.

### Module blocks
 
```tcl
# main.tf
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.19.0"

  name = "example-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-west-2a", "us-west-2b", "us-west-2c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24"]

  enable_dns_hostnames    = true
}
```

### Install module && Plan and apply changes

- **Install module**: When you add a new module to your configuration, you must run the ```terraform init``` command to install it. Terraform automatically installs public modules from the Terraform Registry. 
```sh
terraform init
terraform apply

# Inspect Terraform state
terraform state list

# Your VPC includes two public subnets. Print out the Terraform's state for the first one.
terraform state show module.vpc.aws_subnet.public[0]
```

---

# Destroy Infrastructure
> When you no longer need the infrastructure managed by your workspace, use Terraform to destroy it. Terraform destroys resources when you remove them from your configuration and apply the change. You can also destroy all of the resources managed by your configuration with the terraform destroy command.

## Remove resource

- When you remove a resource from your configuration and create a Terraform plan, Terraform will notice that the configuration for the resource no longer exists, and plan to remove the corresponding infrastructure.

```sh
terraform apply
```

## Destroy workspace
 
```sh
terraform destroy
```
