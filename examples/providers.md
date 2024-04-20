# Provider Configuration

Providers allow Terraform to interact with cloud providers, SaaS providers, and other APIs.

Some providers require you to configure them with endpoint URLs, cloud regions, or other settings before Terraform can use them.

### Syntax
```
provider "<provider_name>" {
  ...
}
```

## Aliases
You can optionally define multiple configurations for the same provider, and select which one to use on a per-resource or per-module basis. The primary reason for this is to support multiple regions for a cloud platform.

---
**NOTE** 

> Using multiple provider blocks with the same provider name, it is **MANDATORY** to use the alias meta-argument to provide an extra name segment.
---

### Syntax
```
provider "<provider_name>" {
  alias  = "<first_alias>"
  ...
}

provider "<provider_name>" {
  alias  = "<second_alias>"
  ...
}

resourse "<resourse1_type>" "<resource1_name>" {
  provider = <provider_name>.<first_alias>
  ...
}

module "<module_name>" {
  source = "<module_path>"
  providers = {
    aws = <provider_name>.<second_alias>
  }
}
```

## Examples

### AWS providers for VPC peering, one accoutn, different regions, auto accept

```
provider "aws" {
  alias  = "east"
  region = "us-east-1"
}

provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

data "aws_caller_identity" "peer" {
  provider = aws.west
}

resource "aws_vpc_peering_connection" "connection" {
  provider      = aws.east
  vpc_id        = aws_vpc.vpc1.id
  peer_region   = "us-west-2"
  peer_owner_id = data.aws_caller_identity.peer.account_id
  peer_vpc_id   = aws_vpc.vpc2.id
  auto_accept   = false

}

resource "aws_vpc_peering_connection_accepter" "accepter" {
  provider                  = aws.west
  vpc_peering_connection_id = aws_vpc_peering_connection.connection.id
  auto_accept               = true
}

resource "aws_vpc" "vpc1" {
  provider   = aws.east
  cidr_block = "10.1.0.0/16"
}

resource "aws_vpc" "vpc2" {
  provider   = aws.west
  cidr_block = "10.2.0.0/16"
}
```

## References
[Terraform documentation](https://www.terraform.io/language/providers/configuration)
[Version constrains](https://developer.hashicorp.com/terraform/language/expressions/version-constraints)
