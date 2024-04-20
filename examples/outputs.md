# Output Values

Output values make information about your infrastructure available on the command line, and can expose information for other Terraform configurations to use. Output values are similar to return values in programming languages.

Output values have several uses:
- A child module can use outputs to expose a subset of its resource attributes to a parent module.
- A root module can use outputs to print certain values in the CLI output after running ```terraform apply``` or ```terraform refresh```.
- When using remote state, root module outputs can be accessed by other configurations via a ```terraform_remote_state``` data source.

Output values are like function return values.

### Syntax
```
output "<output_name>" {
  value       = <output_value>
  description = <output_description> # optional
  sensitive   = <true or false> # optional, by default false
}
```

## Examples

### Output value of an AWS VPC ID Output
```
resource "aws_vpc" "sample" {
  cidr_block = "10.0.0.0/16"
}

output "vpc_id" {
  value = aws_vpc.sample.id
}
```

### Output value of an AWS ARN of an S3 Bucket
```
resource "aws_s3_bucket" "sample_bucket" {
  bucket = "sample-devops-bucket-1bc23"
}

output "bucket_arn" {
  value = aws_s3_bucket.sample_bucket.arn
}
```

### Output value of an Azure Resource Group ID
```
resource "azurerm_resource_group" "sample_group" {
  name     = "sample-group"
  location = "West Europe"
}

output "resource_group_id" {
  value = azurerm_resource_group.sample_group.id
}
```

### Output value of a GCP VPC ID
```
resource "google_compute_network" "gcp_network" {
  name = "gcp-network"
}

output "vpc_id" {
  value = google_compute_network.gcp_network.id
}
```

### Password Generator (plain text)
```
resource "random_id" "generated_id" {
  byte_length = 8
}

output "id" {
    value = random_id.generated_id.id
    sensitive = false
}
```
```
# terraform apply
...
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

id = "7QLNMeEPwOY"
```

### Password Generator (sensitive)
```
resource "random_id" "generated_id" {
  byte_length = 8
}

output "id" {
    value = random_id.generated_id.id
    sensitive = true
}
```
```
# terraform apply
...
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

id = <sensitive>
```

## References
[Terraform documentation](https://www.terraform.io/language/values/outputs)