# Input Variables

Input variables let you customize aspects of Terraform modules without altering the module's own source code. This functionality allows you to share modules across different Terraform configurations, making your module composable and reusable. Input variables works like function arguments.

### Syntax
```
variable "<variable_name>"  {
  type        = <variable_type> # (optional)
  default     = <default_variable_value> # the value will be used if it is not declared (optional)
  description = "<Variable description>" # (optional)
  sensitive   = false   # (optional) if set to true, the output in the terraform plan is masked
  validation {...}      # Validation options for the input
  ...
}

output "variable_output" {
  value = var.<variable_name>
}
```

## Examples

### AWS EC2 Instance Resource with an Instance Type as a variable
```
variable "instance_type" {
  type        = string
  default     = "t2.micro"
  description = "Instance type for the new instance"
}

resource "aws_instance" "simple_instance" {
  ami           = "ami-a1b2c3d4"
  instance_type = var.instance_type
}
```

### SSM Parametr with sensitive

```
variable "instance_password" {
  type        = string
  description = "Service user instance password"
  sensitive   = true
}

resource "aws_ssm_parameter" "secret" {
  name        = "/instance/password"
  description = "Service user instance password"
  type        = "SecureString"
  value       = var.instance_password
}
```
Result
```
# terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_ssm_parameter.secret will be created
  + resource "aws_ssm_parameter" "secret" {
      + arn            = (known after apply)
      + data_type      = (known after apply)
      + description    = "Service user instance password"
      + id             = (known after apply)
      + insecure_value = (known after apply)
      + key_id         = (known after apply)
      + name           = "/instance/password"
      + tags_all       = (known after apply)
      + tier           = (known after apply)
      + type           = "SecureString"
      + value          = (sensitive value)
      + version        = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

### Validation options for the variable AMI ID

```
variable "ami_id" {
  type        = string
  description = "The id of the machine image (AMI) to use for the server."

  validation {
    condition     = length(var.ami_id) > 4 && substr(var.ami_id, 0, 4) == "ami-"
    error_message = "The image_id value must be a valid AMI id, starting with \"ami-\"."
  }
}
```

## References
[Terraform documentation](https://www.terraform.io/language/values/variables)
