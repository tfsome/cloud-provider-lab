# Resource Blocks

Resource block describes one or more infrastructure objects, such as virtual networks, compute instances, or higher-level components such as DNS records. Declaring a resource tells Terraform that it should CREATE and manage the Resource described.

### Syntax
```
resource "<resource_type>" "<resource_name>" {
  <attribute1_name> = <attribute1_value>
  <attribute2_name> = <attribute2_value>
  ...
}
```

## Examples

### AWS EC2 Instance Resource
```
resource "aws_instance" "simple_instance" {
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
}
```

### AWS S3-bucket
```
resource "aws_instance" "simple_bucket" {
  bucket = "epam-devops-bucket-2ja8s5s"
}
```

### Azure Resource Group
```
resource "azurerm_resource_group" "sample_group" {
  name     = "sample-group"
  location = "West Europe"
}
```

### GCP VPC
```
resource "google_compute_network" "gcp_network" {
  name = "gcp-network"
}
```

### Null Resource
```
resource "null_resource" "simple_null" {
  triggers = {
    always_run = timestamp()
  }
}
```

## References
[Terraform documentation](https://www.terraform.io/language/resources)