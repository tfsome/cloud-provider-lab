# Data Sources

Data sources allow Terraform to use information defined outside of Terraform, defined by another separate Terraform configuration, or modified by functions. 
For example: <em>Availability Zones</em>, <em>Account ID</em>, <em>VPC ID</em>, <em>Subnet IDs</em>, <em>Terraform State file</em>, etc.

A data source is declared using a ```data``` block:

### Syntax
```
data "<datasource_type>" "<datasource_name>" {
  <attribute1_name> = <attribute1_value>
  <attribute2_name> = <attribute2_value>
  ...
}

output "datasource_output" {
  value = data.<datasource_type>.<datasource_name>.<fetched_attribute>
}

```

## Examples

### Getting data about AWS Availability zones
```
data "aws_availability_zones" "available" {
  state = "available"
}

output "aws_availability_zones" {
  value = data.aws_availability_zones.available
}
```

### Getting data about AWS Account ID
```
data "aws_caller_identity" "current" {}

output "aws_account_id" {
  value = data.aws_caller_identity.current.account_id
}
```

### Getting data about AWS VPC CIDR Block
```
data "aws_vpc" "sample_vpc" {
  id = "vpc-1f092d52"
}

output "aws_vpc_cidr_block" {
  value = data.aws_vpc.sample_vpc.cidr_block
}
```

### Getting data about latest Amazon Linux AMI ID which belongs to Amazon
```
data "aws_ami" "latest_amazon_linux_ami" {
  owners = ["amazon"]
  most_recent = true

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-ebs"]
  }
}

output "latest_amazon_linux_ami" {
  value = data.aws_ami.latest_amazon_linux_ami.id
}
```


### Getting data from External Remote State from S3 bucket
```
data "terraform_remote_state" "devops_network_tfstate" {
  backend = "s3"

  config = {
    bucket   = "epam-devops-tfstates-1s8o"
    key      = "network/devops-network-tfstate.tfstate"
    region   = "us-east-1"
  }
}

output "aws_subnets_ids" {
  value = data.terraform_remote_state.devops_network_tfstate.outputs.private_subnets_ids
}
```

## References
[Terraform documentation](https://www.terraform.io/language/data-sources)