- [Problem to Be Solved](general_task.md#problem-to-be-solved-in-this-lab)
  * [Explanation of the Solution](general_task.md#explanation-of-the-solution)
  * [PRE-REQUISITES](general_task.md#pre-requisites)
- [Creating Infrastructure](general_task.md#creating-infrastructure)
  * [TASK 1 - Creating Network Resources](general_task.md#task-1---creating-network-resources)
  * [TASK 2 - Create resources for SSH Authentication](general_task.md#task-2---create-resources-for-ssh-authentication)
  * [TASK 3 - Create an Object Storage](general_task.md#task-3---create-an-object-storage)
  * [TASK 4 - Create IAM Resources](general_task.md#task-4---create-iam-resources)
  * [TASK 5 - Configure Network Security](general_task.md#task-5---configure-network-security)
  * [TASK 6 - Form TF Output](general_task.md#task-6---form-tf-output)
  * [TASK 7 - Configure a remote data source](general_task.md#task-7---configure-a-remote-data-source)
  * [TASK 8 - Configure application instances behind a Load Balancer](general_task.md#task-8---configure-application-instances-behind-a-load-balancer)
  * [TASK 9 - Use data discovery](general_task.md#task-9---use-data-discovery)
- [Working with Terraform state](general_task.md#working-with-terraform-state)
  * [TASK 10 - Move resources](general_task.md#task-10---move-resources)
  * [TASK 11 - Move state to other backends](general_task.md#task-11---move-state-to-other-backends)
  * [TASK 12 - Import resources](general_task.md#task-12---import-resources)
- [Advanced tasks](general_task.md#advanced-tasks)
  * [TASK 13 - Expose node output with nginx](general_task.md#task-13---expose-node-output-with-nginx)
  * [TASK 14 - Modules](general_task.md#task-14---modules)


# Problem to Be Solved in This Lab
 This lab shows you how to use Terraform to create infrastructure in cloud environment including compute, network, security and IAM resources. Each cloud compute instance will report its data to a specified object storage on startup. This task is binding to real production needs – for instance, developers could request compute instances with ability to writing debug information to object storage.


### Explanation of the Solution
You will use Terraform with cloud provider to create 2 separate Terraform configurations/states:
 1) Base configuration (Where we'll create some resource required in another configuration)
 2) Compute configuration (Here we'll create autoscaling group with node running behind the Load Balancer)


After you’ve created configuration, we will work on its optimization like using data driven approach and creating modules.

#### Deployment diagram

 ![](./img/lab_diagram.png)


## PRE-REQUISITES
1. Fork current repository. A fork is a copy of a project and this allows you to make changes without affecting the original project.
1. Clone the forked repository to you workstation. All actions should be done under your fork and Terraform gets it context from your local clone working directory:
    - Change current directory to `/cloud-provider-lab/base` folder and create `root.tf` file.
    - Add a `terraform {}` empty block to this file.
    - Create epmty files `variables.tf` and `locals.tf`. These files will be used for variables and local variables.
    - For AWS:
      - Create an AWS provider block inside `root.tf` file with the following attributes:
        - `region = "us-east-1"`
        - `shared_credentials_file = "~/.aws/credentials"`.

        **Hint**: Add your AWS credentials to the `~/.aws/credentials` file. Refer to [this](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) document for details.

    - For GCP:
      - Create a GCP provider block inside `root.tf` file with the following attributes:
        - `project = "{gcp_project_id}"`
        - `credentials = file("~/.gcp/credentials.json"`.

        **Hint**: Add your GCP service account credentials to the `~/.gcp/credentials.json` file. Refer to [this](https://registry.terraform.io/providers/hashicorp/google/latest/docs/guides/getting_started) document for details.

    - For Azure:
      - Create an Azure provider block inside `root.tf` file with the following attributes:
        - `skip_provider_registration = true`
        - `features {}`
      - Install Azure CLI (for Linux) or Azure PowerShell module (for Windows).
      - Perform authentication with command `az login` or `Connect-AzAccount` (for PowerShell)

      **Hint**: There are other ways to authenticate terraform provider before the usage. Refer to [this](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs) document for details.


    Run `terraform init` to initialize your configuration.
    Run `terraform validate` and `terraform fmt` to check if your configuration is valid and fits to a canonical format and style. Do this each time before applying your changes.
    Run `terraform plan` to ensure that there are no changes.

    Please use **underscore** Terraform resources naming, e.g. `my_resource` instead of `my-resource`.


1. Change current directory  to `~/cloud-provider-lab/compute` and repeat the steps in [2].

You are ready for the lab!

# Creating Infrastructure

## TASK 1 - Creating Network Resources
Change current directory  to `~/cloud-provider-lab/base`

Create a network stack for your infrastructure:

### For AWS:

-	**VPC**: `name={SomeName}-01-vpc`, `cidr=10.10.0.0/16`
-	**Public subnets**:
    - `name={SomeName}-01-subnet-public-a`, `cidr=10.10.1.0/24`, `az=a`
    - `name={SomeName}-01-subnet-public-b`, `cidr=10.10.3.0/24`, `az=b`
    - `name={SomeName}-01-subnet-public-c`, `cidr=10.10.5.0/24`, `az=c`
-	**Internet gateway**: `{SomeName}-01-igw`
-	**Routing table to bind IGW with Public subnets**: `name={SomeName}-01-rt`

### For GCP:

-	**VPC**: `name={SomeName}-01-vpc`, `auto_create_subnetworks=false`
-	**Public subnetworks**:
    - `name={SomeName}-01-subnetwork-central`, `cidr=10.10.1.0/24`, `region=us-central1`
    - `name={SomeName}-01-subnetwork-east`, `cidr=10.10.3.0/24`, `region=us-east1`

### For Azure:
- **Resource Group**: `name={SomeName}-01`
-	**Virtual Network**: `name={SomeName}-01-vnet-us-central`, `cidr=10.10.0.0/16`, `location="centralus"`
-	**Subnets**: `name={SomeName}-01-subnet`, `cidr=10.10.1.0/24`

**Hint**: A local value assigns a name to an expression, so you can use it multiple times within a module without repeating it.

Store all resources of this task in the `network.tf` file.
Store all locals in `locals.tf`.
If you use terraform variables, store them in `variables.tf`.

Equip all possible resources with following tags or labels:
  - `Terraform=true`,
  - `Project=epam-tf-lab`
  - `Owner={You name}`

**Note**: Not every cloud resources have `tags` or `labels` property.

Run `terraform validate`  and `terraform fmt` to check if your configuration is valid and fits to a canonical format and style. Do this each time before applying your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

### Definition of DONE:

- Terraform created infrastructure with no errors
- Network resources created as expected (check Cloud WebUI)
- Push *.tf configuration files to git
- [PLACEHOLDER] TBD After terraform test checks will be implemented

## TASK 2 - Create resources for SSH Authentication

Ensure that the current directory is `~/cloud-provider-lab/base`

Create a custom ssh key-pair to access your cloud compute instances:

- Create your ssh key pair [refer to this document](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws)
- Create a `variables.tf` file with empty variable `ssh_key` but with the following description `Provides custom public ssh key`.
- Create a `ssh.tf` file for SSH resources. Use `ssh_key` variable as a public key source.
  ### For AWS:
  - Create `aws_key_pair` resource with the name `epam-tf-ssh-key`.
  ### For GCP:
  - Create `google_compute_project_metadata` resource.
  - Create a metadata item key `shared_ssh_key`, as a value use the SSH public key.
  ### For Azure:
  - Create `azurerm_ssh_public_key` resource with the name `epam-tf-ssh-key`.

  **Note** : Despite the fact that a public SSH key is not a secret, in terms of this lab you should not store it in the repository. The public key should be passed as an environment variable:

    `export TF_VAR_ssh_key="YOUR_PUBLIC_SSH_KEY_STRING"`

  <mark>Never store you secrets inside the code!</mark>

- Run `terraform plan` and observe the output.

Equip all possible resources with following tags or labels:
  - `Terraform=true`
  - `Project=epam-tf-lab`
  - `Owner={You name}`

Run `terraform validate` and `terraform fmt` to check if your configuration is valid and fits to a canonical format and style. Do this each time before applying your changes.

Apply your changes when ready.

### Definition of DONE:

- Terraform created infrastructure with no errors
- All resources created as expected (check Cloud WebUI)
- Push *.tf configuration files to git
- [PLACEHOLDER] TBD After terraform test checks will be implemented

## TASK 3 - Create an Object Storage

Ensure that the current directory is  `~/cloud-provider-lab/base`

Create an object bucket as the storage for your infrastructure:

-	Create file `storage.tf`. Storage resources should be described there.
- Create an object cloud storage resource.
  ### For AWS:
  - Create an S3 bucket. Name this bucket `epam-tf-lab-${random_string.my_numbers.result}` to provide it with a unique name.
  ### For GCP:
  - Create a cloud storage bucket. Name this bucket `epam-tf-lab-${random_string.my_numbers.result}` to provide it with a unique name.
  ### For Azure:
  - Create a storage account. Name this account `epamtflab${random_string.my_numbers.result}` to provide it with a unique name.
  - Create a new storage container with the name `epam-tf-lab-container`

  **Hint** See [random_string](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/string) documentation for details.

-	Set default permissions for the object storage as private. Never share your bucket with the whole world!

Equip all possible resources with following tags or labels:
  - `Terraform=true`
  - `Project=epam-tf-lab`
  - `Owner={You name}`

Run `terraform validate` and `terraform fmt` to check if your configuration is valid and fits to a canonical format and style. Do this each time before applying your changes.
Run `terraform plan` to see your changes.

Apply your changes when ready.

### Definition of DONE:

- Terraform created infrastructure with no errors
- All resources created as expected (check Cloud WebUI)
- Push *.tf configuration files to git
- [PLACEHOLDER] TBD After terraform test checks will be implemented

## TASK 4 - Create IAM Resources
Ensure that the current directory is  `~/cloud-provider-lab/base`

Create IAM resources:
- Create an `iam.tf` file. Create IAM resources there.
  ### For AWS:
  -	**IAM group** (`name={SomeName}-01-group`).
  -	**IAM policy** (`name=write-to-epam-tf-lab`) with write permission for "epam-aws-tf-lab$-{random_string.my_numbers.result}" bucket only.

    **Hint**: store your policy as json document side by side with configurations (or create 'files' subfolder for storing policy) and use templatefile() function to transfer IAM policy with imported S3 bucket name to a resource.
  -	Create **IAM role**, attach the policy to it and create **IAM instance profile** for this IAM role. Allow to assume this role for ec2 service.
  ### For GCP:
  -	Create **Service account** (`account_id={SomeName}-01-account`).
  - Assign the `Storage Object Creator` role to the Service account.
  ### For Azure:
  - **User Assigned Identity** (`name={SomeName}-01`)
  -	**Role** (`name={SomeName}-01`) with write permission for storage account created on the previous stage.
  - Assign the Role to the User Assigned Identity.


Equip all possible resources with following tags or labels:
  - `Terraform=true`
  - `Project=epam-tf-lab`
  - `Owner={Your_name}`

Run `terraform validate`  and `terraform fmt` to check if your configuration is valid and fits to a canonical format and style. Do this each time before applying your changes.
Run `terraform plan` to see your changes.

Apply your changes when ready.

### Definition of DONE:

- Terraform created infrastructure with no errors
- All resources created as expected (check Cloud WebUI)
- Push *.tf configuration files to git
- [PLACEHOLDER] TBD After terraform test checks will be implemented

## TASK 5 - Configure Network Security
Ensure that the current directory is  `~/cloud-provider-lab/base`

Store all resources from this task in the `network_security.tf` file.
Create the following resources:
### For AWS:
-	Security group (`name=ssh-inbound`, `port=22`, `allowed_ip_range="your_IP or EPAM_office-IP_range"`, `description="allows ssh access from safe IP-range"`).
-	Security group (`name=lb-http-inbound`, `port=80`, `allowed_ip_range="your_IP or EPAM_office-IP_range"`, `description="allows http access from safe IP-range to a LoadBalancer"`).
-	Security group (`name=http-inbound`, `port=80`, `source_security_group_id=id_of_lb-http-inbound_sg`, `description="allows http access from LoadBalancer"`).
- Make the most of the `aws_security_group_rule` resource.

**Hint:** source_security_group_id is an attribute of[aws_security_group_rule resource](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule). For details about how to configure securitygroups for loadbalancer see [documentation] (https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-update-security-groups.html)

### For GCP:
-	Firewall rule (`name=ssh-inbound`, `port=22`, `allowed_ip_ranges="your_IP or EPAM_office-IP_ranges"`, `description="allows ssh access from safe IP-range"`, `target_tags=web-instances`).
-	Firewall rule (`name=http-inbound`, `port=80`, `allowed_ip_ranges="130.211.0.0/22", "35.191.0.0/16", "your_IP or EPAM_office-IP_ranges"`, `description="allows http access from LoadBalancer"`, `target_tags=web-instances`).

    **Hint:** These firewall should be created for the VPC which was created in the Task 1

    **Note:** "130.211.0.0/22", "35.191.0.0/16" are IP ranges of the GCP health checkers. See [there](https://cloud.google.com/load-balancing/docs/firewall-rules)

### For Azure:
- Network security group (`name=lab-inbound`).
- For the created network security group create rules:
  -	Network security rule (`name=http-inbound`, `destination_port_range=80`, `source_address_prefix="your_IP or EPAM_office-IP_range"`, `destination_address_prefix="subnet address range"`, `description="allows http access from safe IP-range to the subnet"`).
  -	Network security rule (`name=ssh-inbound`, `destination_port_range=22`, `source_address_prefix="your_IP or EPAM_office-IP_range"`, `destination_address_prefix="subnet address range"`, `description="allows ssh access from safe IP-range"`).
- Assign the NSG to the subnet.

Equip all possible resources with following tags or labels:
  - `Terraform=true`
  - `Project=epam-tf-lab`
  - `Owner={You name}`

Run `terraform validate`  and `terraform fmt` to check if your configuration is valid and fits to a canonical format and style. Do this each time before applying  your changes.
Run `terraform plan` to see your changes.

Apply your changes when ready.

### Definition of DONE:

- Terraform created infrastructure with no errors
- All resources created as expected (check Cloud WebUI)
- Push *.tf configuration files to git
- [PLACEHOLDER] TBD After terraform test checks will be implemented


## TASK 6 - Form TF Output
Ensure that current directory is  `~/cloud-provider-lab/base`

Create outputs for your configuration:

- Create `outputs.tf` file.
### For AWS:
- Following outputs are required: `vpc_id`, `public_subnet_ids`[set of strings], `security_group_id_ssh`, `security_group_id_http`, `security_group_id_http_lb`, `iam_instance_profile_name`, `key_name`, `s3_bucket_name`.
### For GCP:
- Following outputs are required: `vpc_id`, `subnetworks_ids`[set of strings], `service_account_email`, `project_metadata_id`, `bucket_id`.
### For Azure:
- Following outputs are required: `network_name`, `subnet_id`, `network_security_group_id`, `storage_container_name`,`storage_account_name`, `user_managed_identity_id`.

Run `terraform validate`  and `terraform fmt` to check if your configuration is valid and fits to a canonical format and style. Do this each time before applying your changes.
Run `terraform plan` to see your changes.

Apply your changes when ready. You can update outputs without using `terraform apply` - just use the `terraform refresh` command.

### Definition of DONE:

- Push *.tf configuration files to git
- [PLACEHOLDER] TBD After terraform test checks will be implemented

## TASK 7 - Configure a remote data source

Learn about [terraform remote state data source](https://www.terraform.io/docs/language/state/remote-state-data.html).

! Change the current directory to  `~/cloud-provider-lab/compute`
! Copy `root.tf` from `~/cloud-provider-lab/base` to `~/cloud-provider-lab/compute`

Add remote state resources to your configuration to be able to import output resources:

-	Create a data resource for base remote state. (backend="local")

Store all resources from this task in the `data.tf` file.

Run `terraform validate`  and `terraform fmt` to check if your configuration is valid and fits to a canonical format and style. Do this each time before applying your changes.
Run `terraform plan` to see your changes.

Apply your changes when ready.

### Definition of DONE:

- Push *.tf configuration files to git
- [PLACEHOLDER] TBD After terraform test checks will be implemented

## TASK 8 - Configure application instances behind a Load Balancer

**Note**: In this task we are going to join all resources created before to run a compute node using template with a start-up script. See [deployment diagram](#deployment-diagram).


Ensure that the current directory is  `~/cloud-provider-lab/compute`.

Store all resources from this task in the `application.tf` file.

Create required resources:
- Author an init bash script which should get 2 parameters on compute instance start-up and send it to a cloud object storage as a text file with compute_instance_id as its name. Message template:
  ```
  This message was generated on instance {COMPUTE_INSTANCE_ID} with the following UUID {COMPUTE_MACHINE_UUID}
  ```
  Getting Compute Instance Metadata:
  ```
  COMPUTE_MACHINE_UUID=$(cat /sys/devices/virtual/dmi/id/product_uuid |tr '[:upper:]' '[:lower:]')
  COMPUTE_INSTANCE_ID=$(replace this text with request instance id from metadata e.g. using curl. Note that COMPUTE_INSTANCE_ID may have different name on different providers.)
  ```

### For AWS:
- Create a Launch Template resource (`name=epam-tf-lab`, `image_id="actual Amazon Linux AMI2 image id"`, `instance_type=t2.micro`, `security_group_id={ssh-inbound-id,http-inbound-id}`, `key_name`, `iam_instance_profile`, `delete_on_termination = true`(for network interface),  `user_data script`)
- Create an `aws_autoscaling_group` resource (`name=epam-tf-lab`, `max_size=min_size=1`, `launch_template=epam-tf-lab`)
- Create an Application Loadbalancer ( `name=elb-epam-tf-lab`) and attach it to an auto-scaling group with `aws_autoscaling_attachment`. Configure `aws_autoscaling_group` to ignore changes to the `load_balancers` and `target_group_arns` arguments within a lifecycle configuration block (lb_port=80, instance_port=80, protocol=http, `security_group_id={lb-http-inbound-id}`).

**Note:** Please keep in mind that AWS autoscaling group requires using a special format for `Tags` section!

### For GCP:
- Create an Instance Template resources for subnetworks' regions (`name=epam-tf-lab-{region}`, `source_image="debian-cloud/debian-10"`, `machine_type=f1-micro`, `tags="web-instances"`, `service_account`, `startup-script`). Public SSH-key should be fetched from the Project Metadata.
- Create an `google_compute_region_instance_group_manager` resource (`name=epam-gcp-tf-lab-{region}`, `target_size=1`, `instance_template=epam-tf-lab-{region}`)
- Create all required resources to configure a Global HTTP Loadbalancer and attach it to an instance group with `google_compute_health_check`.

### For Azure:
- Create `azurerm_public_ip` resource (`name="epam-tf-lab"`)
- Create `azurerm_lb` resource (`name="epam-tf-lab"`, with `frontend_ip_configuration` with reference on the created public IP address)
- Create `azurerm_lb_backend_address_pool` resource  with reference on the created load balancer
- Create an `azurerm_linux_virtual_machine_scale_set` resource. (`name="epam-tf-lab"`,`instances=2`,`sku="Standard_F2"`, `custom_data=base64encode("{init_script_file}")`, source_image_reference with `publisher="Canonical"`, `offer="0001-com-ubuntu-server-focal"`, `sku="20_04-LTS"`, `version="latest"`). The network interface of future virtual machines should be in the previously created subnet with the previosly created network security group. Authentication should be through created SSH key.
- Create `azurerm_lb_probe` (`name="epam-tf-lab"`, `protocol="Http"`, `port=80`, `request_path="/"`)
- Create `azurerm_lb_rule` (`protocol="Tcp"`, `frontend_port=80`, `backend_port=80`) with reference to the previosly created frontend IP configuration name, IDs of the backend address pool and probe ID.


Equip all possible resources with following tags or labels:
  - `Terraform=true`
  - `Project=epam-tf-lab`
  - `Owner={You name}`

Run `terraform validate` and `terraform fmt` to check if your configuration valid and fits to a canonical format and style. Do this each time before applying your changes.
Run `terraform plan` to see your changes.

Apply your changes when you're ready.

As a result, each time a cloud compute instance launches a new file should be created in your cloud object storage.

### Definition of DONE:

- Terraform created infrastructure with no errors
- All resources created as expected (check Cloud WebUI)
- After a new instance launch, a new text file appears in the cloud object storage with the appropriate text.
- Push *.tf configuration files to git
- [PLACEHOLDER] TBD After terraform test checks will be implemented

## TASK 9 - Use data discovery

Learn about [terraform data sources](https://www.terraform.io/docs/language/data-sources/index.html) and [querying terraform data sources](https://learn.hashicorp.com/tutorials/terraform/data-sources?in=terraform/configuration-language&utm_source=WEBSITE&utm_medium=WEB_BLOG&utm_offer=ARTICLE_PAGE).

In this task we are going to use a data driven approach instead to use remote state data source. This approach could be more flexible and remove dependency between states.

#### base configuration
Change current directory to `~/cloud-provider-lab/base`

Refine your configuration :
- Use a data source to request
  ### For AWS:
  - An account ID and region for the provider.
  ### For GCP:
  - A project numeric ID and default service account email.
  ### For Azure:
  - A Subscription ID and a Client ID of the current user.

Store all resources from this task in the `data.tf` file.

Refer to this data sources in your in case it works. E.g. Azure Subscription ID is required for most resources but not AWS Accound Id.

#### compute configuration
Change the current directory to `~/cloud-provider-lab/compute`

Refine your configuration:

- Use a data source to request resource group created in the `~/cloud-provider-lab/base` and assign it to your resources.

### For AWS:
- Following data sources are required: `vpc_id`, `public_subnet_ids`[set of strings], `security_group_id_ssh`, `security_group_id_http`, `security_group_id_http_lb`; As for the `iam_instance_profile_name`, `key_name` and  `s3_bucket_name` - they are contain simple text and could be strictly defined with locals without calling data sources.
### For GCP:
- Following data sources are required: `vpc_id`, `subnetworks_ids`[set of strings], `project_metadata_id`, `bucket_id`. As for the `service_account_email` - it's contains simple text and could be strictly defined with locals without calling data sources.
### For Azure:
- Following data sources are required: `subnet_ids`[set of strings], `network_security_group_id`, `user_managed_identity_id`. As for the `network_name`,`storage_container_name` and `storage_account_name`, - they are contain simple text and could be strictly defined with locals without calling data sources.


Hint: These data sources should replace remote state outputs, therefore you can delete `data "terraform_remote_state" "base"` resource from your current state and the `outputs.tf` file from the `base` configuration. **Don't forget to replace references with a new data sources.**

Hint: run `terraform refresh` command under `base` configuration to reflect changes.

Store all resources from this task in the `data.tf` file.

Run `terraform validate` and `terraform fmt` to check if your configuration is valid and fits to a canonical format and style. Do this each time before applying your changes.
Run `terraform plan` to see your changes. Also you can use `terraform refresh`.
If applicable all resources should be defined with the provider alias.

Apply your changes when ready.

# Working with Terraform state

In this section, we will delve into Terraform state management and Terraform backends. A backend serves as the repository where Terraform  stores its state files. The default backend is local, meaning that it stores the state file on your local disk. To enable effective collaboration with your colleagues, you should consider utilizing a [remote backend](https://developer.hashicorp.com/terraform/language/settings/backends/remote).

Switching the backend is a straightforward process accomplished by [re-initializing](https://developer.hashicorp.com/terraform/cli/commands/init#backend-initialization) with Terraform.

It's crucial to highlight an important note regarding importing or moving resources between states in the upcoming tasks. Typically, there are several methods for transferring resources between two states:

 1)   Employing the built-in `terraform state mv` command.
 2)   Directly editing state files.
 3)   Using the `terraform import` command.

When migrating a resource block (your code) between configurations, you must simultaneously transfer the data between states. Additionally, there is an [experimantal feauture](https://developer.hashicorp.com/terraform/language/import/generating-configuration) available for auto-generating configuration during the import process. However, this is beyond the scope of our current training.

Please note that the `terraform state mv` [1] command exclusively functions with local states. Therefore, if you opt for this method, ensure that you migrate your resources before transitioning to a remote state.

Alternatively, you can copy state files locally and relocate resources using either `terraform state mv`` or inline editing[2]. However, please be aware that inline editing is not a secure option, and you would need to modify the 'serial' counter each time you make changes in the state file.

Finally, you have the option to import an existing resource into a new state using the `terraform state import resource.name resource.id` command[3] and remove it from the old state with `terraform state rm resource.name`.

Any of these options will serve the purpose, but for educational purposes, we have opted for the simplest approach using `terraform state mv` command.



## TASK 10 - Move resources

Learn about [terraform state mv](https://www.terraform.io/docs/cli/commands/state/mv.html) command

You are going to move previously created resource from the [Task 2] from `base` to `compute` state.
Hint: Keep in mind that there are 3 instances: cloud resource, Terraform state file which store some state of that resource, and Terraform configuration which describe resource. "Move resource" is moving it between states. Moreover to make it work you should delete said resource from source configuration and add it to the destination configuration (this action is not automated).

- Move the resource created in the Task 2 from the `base` state to the `compute` using `terraform state mv` command:
  ### For AWS:
  - The `epam-tf-ssh-key` AWS Key Pair resource.
  ### For GCP:
  - The `epam-tf-ssh-key` Project Metadata resource.
  ### For Azure:
  - The `epam-tf-ssh-key` Azure SSH Public Key resource.

- Update both configurations according to this move.
- Run `terraform plan` on both configurations and observe the changes. Hint: there should not be any changes detected (no resource creation or deletion in case of correct resource move).

Run `terraform validate` and `terraform fmt` to check if your configuration is valid and fits to a canonical format and style.

### Definition of DONE:

- Terraform moved resources with no errors
- All resources are NOT changed (check Cloud Web UI)

## TASK 11 - Move state to other backends

Learn about terraform backends [here](https://developer.hashicorp.com/terraform/language/settings/backends/configuration)

### For AWS:
- Create an S3 Bucket(`name=epam-aws-tf-state-${random_string}`) and a DynamoDB table as a pre-requirement for this task. There are multiple ways to do this, including Terraform and CloudFormation. But please just create both resources by a hands in AWS console. Those resources will be out of our IaC approach as they will never be recreated.

  Learn about [terraform backend in AWS S3](https://www.terraform.io/docs/language/settings/backends/s3.html)

### For GCP:
- Create a Cloud Storage Bucket(`name=epam-gcp-tf-state-${random_string}`) as a pre-requirement for this task. Please create the resource by a hands in GCP console. That resource will be out of our IaC approach as it will never be recreated.

  Learn about [terraform backend in Cloud Storage Bucket](https://developer.hashicorp.com/terraform/language/settings/backends/gcs)

### For Azure:
- Create a new storage account (`name=epamazurelabstate${random_string}`) and then a storage container (`name=epam-azure-tf-state`). There are multiple ways to do this, including Terraform and ARM. But please just create both resources by a hands in Azure Portal. Those resources will be out of our IaC approach as they will never be recreated.

  Learn about [terraform backend in Azure storage container](https://developer.hashicorp.com/terraform/language/settings/backends/azurerm)

Refine your configurations:

- Refine `base` configuration by moving local state to an appropriate available state.
- Refine `compute` configuration by moving local state to an appropriate available state.

Do not forget to change the path to the remote state for `compute` configuration.

Run `terraform validate` and `terraform fmt` to check if your modules valid and fits to a canonical format and style.
Run `terraform plan` to see your changes and re-apply your changes if needed.

## TASK 12 - Import resources

Learn about the [terraform import](https://www.terraform.io/docs/cli/import/index.html) command.

You are going to create and import a new resource (IAM resources) to your state.
Hint: Keep in mind that there are 3 instances: cloud resource, Terraform state file which store some state of that resource, and Terraform configuration which describe resource. "Importing a resource" is importing its attributes into a Terraform state. Then you have to add said resource to the destination configuration (this action is not automated).

- Create an IAM resource in Cloud WebUI:
  ### For AWS:
  - An IAM Role with the name `test-import`.
  ### For GCP:
  - An IAM Service account with the name `test-import`.
  ### For Azure:
  - A User Managed Identity with the name `test-import`.

- Add a new resource with the name `test-import` to the `compute` configuration.
  ### For AWS:
  - `aws_iam_role`
  ### For GCP:
  - `google_service_account`
  ### For Azure:
  - `azurerm_user_assigned_identity`

- Run `terraform plan` to see your changes but do not apply changes.
- Import IAM Resource with the name `test-import` to the `compute` state.
- Run `terraform plan` again to ensure that import was successful.

Run `terraform validate` and `terraform fmt` to check if your configuration is valid and fits to a canonical format and style.
If applicable all resources should be tagged with following tags:
- `Terraform=true`,
- `Project=epam-tf-lab`.

If applicable all resources should be defined with the provider alias.

- Terraform imported resources with no errors
- All resources are NOT changed (check Cloud Web UI)


# Advanced tasks

## TASK 13 - Expose node output with nginx

Ensure that the current directory is  `~/cloud-provider-lab/compute`

Change init script in the task 8 as follows:

-   Nginx binary should be installed on instance (`port=80`).
-   Environmnet variables `COMPUTE_INSTANCE_ID` and `COMPUTE_MACHINE_UUID` should be defined (see Task 8).
-   Nginx default page should be configured to return the same text as we put previously into the cloud object storage in Task 8: "This message was generated on instance ${COMPUTE_INSTANCE_ID} with the following UUID ${COMPUTE_MACHINE_UUID}".
-   Nginx server should be started.

Run `terraform validate` and `terraform fmt` to check if your configuration is valid and fits to a canonical format and style. Do this each time before applying your changes.
Run `terraform plan` to see your changes.

Apply your changes when ready.

### Definition of DONE:

- Terraform created infrastructure with no errors
- All resources are NOT changed (check Cloud WebUI)
- Nginx server responds on Loadbalancer's URL (for AWS) or IP Address (for GCP and Azure) with expected response.

## TASK 14 - Modules

Learn about [terraform modules](https://www.terraform.io/docs/language/modules/develop/index.html)

Refine your configurations:

- Refine `base` configuration by creating module for network related resources.
- Refine `base` configuration by creating module for network security related resources.
- Refine `base` configuration by creating module for IAM related resources.
- [Optional] Refine `compute` configuration by creating application resources behind a Load Balancer.


Store your modules in `~/cloud-provider-lab/modules/` subfolders.

Run `terraform validate` and `terraform fmt` to check if your modules are valid and fit to a canonical format and style.
Run `terraform plan` to see your changes and re-apply your changes if needed.
