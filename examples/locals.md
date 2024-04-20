# Local Values

A local value assigns a name to an expression, so you can use the name multiple times within a module instead of repeating the expression.
Local values are like a function's temporary local variables.

The most actual and effective way to use local variables is when it is required to calculate some value based on the values of input variables, data source's attributes, or resource's attributes.

### Syntax
```
locals {
  <local_variable_name> = <local_variable_value>
}

output "local_output" {
  value = local.<local_variable_name>
}
```

## Examples

### Calculate Instance tags basing on the common tags
```
variable "common_tags" {
  type = map
  description = "Set of the common tags for the module"
}

locals {
  instance_tags = merge(var.common_tags, 
    {
        "Name" = "Web Server"
    })
}

resource "aws_instance" "web_server" {
  ami           = "ami-a1b2c3d4"
  instance_type = var.instance_type
  tags = local.instance_tags
}
```

### Calculate the S3 bucket name which is based on the project name and environment
```
variable "project_name" {
  type        = string
  default     = "Terraform Learn"
  description = "The name of the project"
}

variable "environment" {
  type = string
  default = "Dev"
  description = "The name of the project"
}

locals {
  project_name = replace(lower(var.project_name), " ", "-")
  environment = lower(var.environment)
  bucket_prefix = "${local.project_name}-${local.environment}"
}

resource "aws_s3_bucket" "my_s3_bucket" {
  bucket_prefix = local.bucket_prefix
  acl    = "private"
}
```

## References
[Terraform documentation](https://www.terraform.io/language/values/locals)