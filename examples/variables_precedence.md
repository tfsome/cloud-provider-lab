# Terraform Variables Precedence

There are several ways to define and re-define variables in Terraform. If the same variable is assigned multiple values, Terraform uses the last value it finds, overriding any previous values. Note that the same variable cannot be assigned multiple values within a single source.

Lets look on possible options from the less precedent values to the most one.

For the example we will use simple module that only returns the `name` output. Module itself will be unchanged during our experiments.

```
output "result_name" {
  value = var.name
}

variable "name" {
  default = "batman"
}

```


## 10. Module defaults

Lets call this module from our code with the default values:

```
module "my_name"{
  source = ../modules/my_name
}

output "result" {
  value = module.my_name.result_name
}

```

`terraform plan` returns “batman” as defined in its defaults:

```
Changes to Outputs:
  + result = "batman"
```

## 9. Interactive

Now lets call the module and redefine the value of module variable `name` with  `another_name` but not provide any default value for it.

```
module "my_name" {
  source = "../modules/my_name"
  name   = var.another_name
}

variable "another_name" {
}

output "result" {
  value = module.my_name.result_name
}

```

`terraform plan` asks for the value:


```
var.another_name
  Enter a value: spiderman

  ...
Changes to Outputs:
  + result = "spiderman"

```

## 8. Variables defaults

Lets define our variable:

```
variable "another_name" {
  default = "superman"
}
```


And `terraform plan` returns


```
Changes to Outputs:
  + result = "superman"
```

## 7. Environment variables

Now we have to set environment variable before run `terraform plan`:

`export TF_VAR_another_name="wolverine"`

And `terraform plan` returns


```
Changes to Outputs:
  + result = "wolverine"
```

## 6. terraform.tfvars

All values defined in external file with `terraform.tfvars` will be auto loaded and override any less precedent values.

`another_name = "storm"`

And `terraform plan` returns


```
Changes to Outputs:
  + result = "storm"
```

## 5. terraform.tfvars.json

The same as previous but in json format:

```
{
  "another_name": "thor"
}

```


And `terraform plan` returns


```
Changes to Outputs:
  + result = "thor"
```
This will return “thor” even `terraform.tfvars` exists with another value.

## 4. *auto.tfvars.*

Precedence is the folowing here:

`somename.auto.tfvars.json` overrides `somename.auto.tfvars`.

## 3. Command line

Lets try to run `terraform plan -var-file=this_name_wins.ini`

This returns another name from the external file. Terraform ignores file extension.
Comand line variable overrides any previous in scope of `terraform plan` or `terraform apply` commands. This method uses Terraform Enterprise to load variables to its workspaces. 

## 2. terraform test variables

`terraform test` let authors validate that module configuration updates do not introduce breaking changes. For testing purposes we could need to re-define variables.
This could be done in special section of *.tftest.hcl

```
variables {
  another_name = "test_name_1"
}

run "uses_root_level_value" {

  command = plan

  assert {
    condition     = var.another_name == "test_name_1"
    error_message = "Name didn't match"
  }

}

```

## 1. terraform test variables in run block

`terraform test` has another level to re-define variables in `run` block:

```
variables {
  another_name = "test_name_1"
}

run "uses_root_level_value" {

  command = plan

  variables {
    another_name = "test_name_2"
  }

  assert {
    condition     = var.another_name == "test_name_1"
    error_message = "Name didn't match"
  }

}

```

And test will failed as we re-define the value that not match to our condition:

```
terraform test  -verbose
name_test.tftest.hcl... in progress
  run "uses_root_level_value"... fail

Changes to Outputs:
  + result = "test_name_2"

You can apply this plan to save these new output values to the Terraform state, without changing any real infrastructure.

╷
│ Error: Test assertion failed
│ 
│   on name_test.tftest.hcl line 14, in run "uses_root_level_value":
│   14:     condition     = var.another_name == "test_name_1"
│     ├────────────────
│     │ var.another_name is "test_name_2"
│ 
│ Name not expected
╵
name_test.tftest.hcl... tearing down
name_test.tftest.hcl... fail

Failure! 0 passed, 1 failed.
```

