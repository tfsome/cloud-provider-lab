# Terraform Settings

The special terraform configuration block type is used to configure some behaviors of Terraform itself, such as requiring a minimum Terraform version to apply your configuration.

### Syntax 

Terraform settings are gathered together into terraform blocks:
```
terraform {
  # ...
}
```
---
**NOTE** 

> Within a terraform block, only constant values can be used; arguments may not refer to named objects such as resources, input variables, etc, and may not use any of the Terraform language built-in functions.
---

## Examples

### Setting the terraform version 1.2.x
```
terraform {
  required_version = "~> 1.2.0"
}
```

### Setting the AWS provider version between 4.0 and 4.10
```
terraform {
  required_providers {
    aws = {
      version = ">= 4.0, <= 4.10"
    }
  }
}
```

### Backend configuration (local)
```
terraform {
  backend "local" {
    # a state file will be named project.tfstate in /home/user/project/
    path = "/home/user/project/project.tfstate"
  }
}
```

### Backend configuration (AWS S3 + DynamoDB lock)
```
terraform {
  backend "s3" {
    bucket         = "s3-bucket-for-state-files"
    key            = "project/project.tfstate"
    region         = "eu-west-1"
    encrypt        = "true"
    dynamodb_table = "dynamo-table-for-lock"
  }
}
```

### Backend configuration (Terraform Cloud)
```
terraform {
  backend "remote" {
    hostname = "app.terraform.io"
    organization = "company"

    workspaces {
      name = "my-app-prod"
    }
  }
}
```

### Backend configuration (AzureRM)
```
terraform {
  backend "azurerm" {
    resource_group_name  = "StorageAccount-ResourceGroup"
    storage_account_name = "abcd1234"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

### Backend configuration (Google Cloud Storage)
```
terraform {
  backend "gcs" {
    bucket  = "tf-state-prod"
    prefix  = "terraform/state"
  }
}
```

## References
[Terraform documentation](https://www.terraform.io/language/settings)
[Version constrains](https://developer.hashicorp.com/terraform/language/expressions/version-constraints)
