# F5-XC-AWS-TGW-SNODE

This repository consists of Terraform templates to bring up a f5-xc-aws-tgw-snode object.

## Usage

- Clone this repo with: `git clone --recurse-submodules https://github.com/cklewar/f5-xc-aws-tgw-snode`
- Enter repository directory with: `cd f5-xc-aws-tgw-snode`
- Obtain F5XC API certificate file from Console and save it to `cert` directory
- Pick and choose from below examples and add mandatory input data and copy data into file `main.tf.example`.
- Rename file __main.tf.example__ to __main.tf__ with: `rename main.tf.example main.tf`
- Initialize with: `terraform init`
- Apply with: `terraform apply -auto-approve` or destroy with: `terraform destroy -auto-approve`

## f5-xc-aws-tgw-snode module usage example

````hcl
variable "project_prefix" {
  type        = string
  description = "prefix string put in front of string"
  default     = "f5xc"
}

variable "project_suffix" {
  type        = string
  description = "prefix string put at the end of string"
  default     = "01"
}

variable "f5xc_api_p12_file" {
  type = string
}

variable "f5xc_api_url" {
  type = string
}

variable "f5xc_api_token" {
  type = string
}

variable "f5xc_tenant" {
  type = string
}

variable "f5xc_namespace" {
  type    = string
  default = "system"
}

variable "f5xc_aws_cred" {
  type    = string
  default = "ck-aws-01"
}

variable "f5xc_aws_region" {
  type    = string
  default = "us-east-2"
}

variable "f5xc_aws_availability_zone" {
  type    = string
  default = "a"
}

variable "owner_tag" {
  type    = string
  default = "c.klewar@f5.com"
}

variable "ssh_public_key_file" {
  type = string
}

locals {
  aws_availability_zone = format("%s%s", var.f5xc_aws_region, var.f5xc_aws_availability_zone)
  custom_tags           = {
    Owner        = var.owner_tag
    f5xc-tenant  = var.f5xc_tenant
    f5xc-feature = "f5xc-aws-vpc-site"
  }
}

provider "volterra" {
  api_p12_file = var.f5xc_api_p12_file
  url          = var.f5xc_api_url
  alias        = "default"
}

provider "aws" {
  region = var.f5xc_aws_region
  alias  = "us-east-2"
}

module "aws_tgw_single_node_new_tgw_new_subnets" {
  source                          = "./modules/f5xc/site/aws/tgw"
  f5xc_tenant                     = var.f5xc_tenant
  f5xc_api_url                    = var.f5xc_api_url
  f5xc_aws_cred                   = var.f5xc_aws_cred
  f5xc_api_token                  = var.f5xc_api_token
  f5xc_namespace                  = var.f5xc_namespace
  f5xc_aws_region                 = var.f5xc_aws_region
  f5xc_aws_tgw_name               = format("%s-tgw-snode-nsnet-%s", var.project_prefix, var.project_suffix)
  f5xc_aws_tgw_no_worker_nodes    = true
  f5xc_aws_default_ce_sw_version  = true
  f5xc_aws_default_ce_os_version  = true
  f5xc_aws_tgw_total_worker_nodes = 0
  f5xc_aws_tgw_primary_ipv4       = "192.168.168.0/21"
  f5xc_aws_tgw_az_nodes           = {
    node0 : {
      f5xc_aws_tgw_workload_subnet = "192.168.168.0/26", f5xc_aws_tgw_inside_subnet = "192.168.168.64/26",
      f5xc_aws_tgw_outside_subnet  = "192.168.168.128/26", f5xc_aws_tgw_az_name = "us-east-2a"
    },
    node1 : {
      f5xc_aws_tgw_workload_subnet = "192.168.169.0/26", f5xc_aws_tgw_inside_subnet = "192.168.169.64/26",
      f5xc_aws_tgw_outside_subnet  = "192.168.169.128/26", f5xc_aws_tgw_az_name = "us-east-2a"
    },
    node2 : {
      f5xc_aws_tgw_workload_subnet = "192.168.170.0/26", f5xc_aws_tgw_inside_subnet = "192.168.170.64/26",
      f5xc_aws_tgw_outside_subnet  = "192.168.170.128/26", f5xc_aws_tgw_az_name = "us-east-2a"
    }
  }
  ssh_public_key = file(var.ssh_public_key_file)
  custom_tags    = {
    Deployment = format("%s-tgw-m-node-new-sub-%s", var.project_prefix, var.project_suffix)
    TTL        = -1
    Owner      = "c.klewar@f5.com"
  }
  providers = {
    aws      = aws.us-east-2
    volterra = volterra.default
  }
}

output "f5xc_aws_vpc_single_node_single_nic_new_vpc_new_subnet" {
  value = module.aws_tgw_single_node_new_tgw_new_subnets.f5xc_aws_tgw
}

module "f5xc_aws_vpc_single_node_single_nic_new_vpc_new_subnet_apply_timeout_workaround" {
  source         = "./modules/utils/timeout"
  depend_on      = module.aws_tgw_single_node_new_tgw_new_subnets.f5xc_aws_tgw
  create_timeout = "1800s"
}
````