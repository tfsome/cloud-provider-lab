# Modules

Modules are containers for multiple resources that are used together. A module consists of a collection of .tf and/or .tf.json files kept together in a directory.
Modules are the main way to package and reuse resource configurations with Terraform.

Types of modules:
- A Root Module - a module which consists of the resources defined in the .tf files in the main working directory.
- A child module - a module which is called by another module (for example from a root module). It can be called multiple times.

If you're familiar with traditional programming languages, it can be useful to compare Terraform modules to function definitions:

- Input variables are like function arguments.
- Output values are like function return values.
- Local values are like a function's temporary local variables.

### Syntax (calling a module)
```
module "<module_name>" {
  source            = <module_path>
  <attribute1_name> = <attribute2_value>
}

output "module_output" {
    value = module.<module_name>.<module_output_name>
}
```
## Examples

### Code of the simple module (AWS VPC)

```
Files
|-vpc
  |- versions.tf
  |- variables.tf
  |- main.tf
  |- outputs.tf
```
versions.tf
```
terraform {
  required_version = "~> 1.2.0"
  required_providers {
    aws = {
      version = "~>4.0"
    }
  }
}
```
variables.tf
```
variable "cidr_block" {
  type        = string
  default     = "10.0.0.0/16"
  description = "A CIDR block for VPC."
}
```
main.tf
```
resource "aws_vpc" "sample" {
  cidr_block = var.cidr_block
}
```
outputs.tf
```
output "vpc_id" {
  value       = aws_vpc.sample.id
  description = "ID of the created VPC"
}
```

### Declare the local VPC module
```
module "vpc" {
  source     = "./vpc"
  cidr_block = "172.16.0.0/24"
}

output "vpc_id" {
  value = module.vpc.vpc_id
}
```

### Declare the VPC module from GitHub (Tag 0.0.1, repository - devops/devops-aws-vpc)
```
module "vpc" {
  source     = "git@github.com:devops/devops-aws-vpc.git?ref=0.0.1"
  cidr_block = "172.16.0.0/24"
}
```

### Declare the VPC module from S3 bucket (S3-bucket - devops-s3-bucket-1hs5, Key - modules/devops-aws-vpc.zip, region - eu-west-1)
```
module "vpc" {
  source     = "devops-s3-bucket-1hs5.s3-eu-west-1.amazonaws.com/modules/devops-aws-vpc.zip"
  cidr_block = "172.16.0.0/24"
}
```

## References
[Terraform documentation](https://www.terraform.io/language/modules/develop)
[Version constrains](https://developer.hashicorp.com/terraform/language/expressions/version-constraints)
