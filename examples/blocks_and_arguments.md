# Arguments and Blocks

There are two core constructs in HCL: arguments and blocks.

An argument assigns a value to a particular name. Values could be any of value types or expression.


A block is a container for other content, like nested blocks and arguments. A block has type and labels(zero, one or two labels in Terraform).

Example 1:

```
    resource "aws_instance" "foo" {                         # block with `type=resource` and 2 labels

      ami           = "ami-005e54dee72cc1d00"               # argument
      instance_type = "t2.micro"                            # argument
    
      network_interface {                                   # nested block with `type=network_interface` and no labels
        network_interface_id = aws_network_interface.foo.id # argument of nested block
        device_index         = 0                            # argument of nested block
      }

      tags = {                                              # map of arguments
          "Name"    = "node-${var.name}-${var.region}"      # nested argument with expression
          "Project" = var.project                           # nested argument
        }
    }
```
Example 2:

```
terraform {                                                 # block with `type=terraform` but without labels
  backend "remote" {                                        # nested block with `type=backend` and 1 label
    hostname = "terraform.example.com"                      # argument of nested block
  }

  required_version = ">= 1.6.2"                             # nested argument

  required_providers {                                      # nested block with `type=required_providers` and no labels
    aws = {                                                 # map of arguments
      source  = "hashicorp/aws"                             # nested argument
      version = "5.31.0"                                    # nested argument
    }
  }
}
```

