<br><br>
![ˈtɛrəfɔːm](https://i.imgur.com/RXAzrGo.jpg)


## Content:

  * [What is IaC and Terraform?](concepts.md#what-is-infrastructure-as-code-iac-and-terraform)
  * [Core concepts](concepts.md#core-concepts)
  * [Terraform Code Snippets](concepts.md#terraform-code-snippets)
  * [Terraform in 15 minutes](concepts.md#terraform-in-15-minutes)
  * [Useful links](concepts.md#useful-links)
  * [Check yourself](concepts.md#check-yourself)
  * [Terraform best practices](concepts.md#terraform-best-practices)
  * [Certification and Trainings](concepts.md#certification-and-trainings)

<br><br>


## What is Infrastructure as Code (IaC) and Terraform?

Infrastructure as code is a mainstream practice of working with cloud providers. In a nutshell, it allows engineers to effectively manage their cloud resources using code. Terraform is one of the most popular (and not without reason!) IaC tools available today. It allows to declare the infrastructure using simple and readable code, which the tool itself then uses to provision all described resources in a necessary order.

Terraform was created by Hashicorp, and is meant to *build and change infrastructure safely and efficiently* as declared in its mission statement by Mitchell Hashimoto, one of Hashicorp's founders. The tool supports provisioning resources using a multitude of providers, but it's not limited to clouds like AWS, GCP or Azure; it also supports working with Kubernetes, Vault, Helm, Consul, VMware vSphere and [dozens](https://registry.terraform.io/browse/providers) of other providers.

To sum it up, Terraform is a powerful, flexible and constantly developing IaC tool successfully used by a community of thousands of professionals around the world.

**Why IaC?**

IaC is a very easy sell. Its main advantages are:

- Speed: resources like whole Kubernetes clusters can be created in mere minutes;
- Consistency: version control and code validation ensure smooth deployment;
- Scalability: from single security group rules to giant EKS clusters;
- Costs: IaC efficiency and scalability allow to optimize infrastructure better);
- Security: secure configurations are enforced and can't be changed in the immutable infrastructure paradigm.

**Why Terraform?**

There are many other IaC tools. Why use Terraform specifically? Apart from the aforementioned wide choice of providers (and thus the ability to manage multi-cloud infrastructure using just one tool), it's also open source. Similar tools like **AWS CloudFormation**, **Azure Resource Manager** and **Google Cloud Deploy** only work with their corresponding cloud environments.

Don't confuse Terraform with *configuration management tools* like **Ansible**, **Chef**, **Puppet** or **SaltStack**. These work with traditional **mutable** infrastructure that continually changes (e.g. an admin uses SSH to change something in an already provisioned EC2 instance or a security group) and is susceptible to configuration drift. These tools also often need a master server and agent installation. Terraform is designed to work with **immutable** infrastructure which (generally) doesn't change after provisioning and conveniently doesn't require any of that. There is a [great article](https://www.hashicorp.com/resources/what-is-mutable-vs-immutable-infrastructure) explaining immutable approach.

Another important Terraform characteristic is **idempotency**. In this context it basically means that infrastructure, once provisioned, won't be changed by running the same provision command again.

Finally, Terraform uses Hashicorp Configuration Language (HCL), which is designed to be simple to learn and to read. HCL is far more flexible and convenient than JSON, YAML or XML.


![errored.tfstate](https://i.imgur.com/6djFpKQ.jpg)

## Core concepts

**Configuration files**

Terraform configuration files (or simply configuration) is a complete document written in the HCL language that tells Terraform how to manage a given collection of infrastructure. A configuration can consist of multiple files and directories. It is important to remember that it represents the **desired state** of resources and does not represent the actual **provisioned resources**.

 [Here](examples/blocks_and_arguments.md) you can find quick overview of HCL base concepts. You will learn more about HCL and it syntax later.

**Terraform state**

The most important concept to grasp for beginners. Think of [Terraform state](https://www.terraform.io/language/state) as a special database where the tool stores what it knows about your provisioned resources. State allows Terraform to know when and where to make changes described in configuration files. **Terraform state**, **provisioned resources** and **configuration files** are crucial to understand. Always remember those three separate entities that Terraform works with and why it works with them.

What happens when you use commands like `terraform apply` or `terraform destroy`? The tool reconciles the desired state (configuration files with your code) with actual resources (in AWS in our case) and, if necessary, makes changes, modifying the state so it reflects the actual resources at the moment of successfully running the command.

Terraform state can be stored locally (not recommended if you're not working alone and for security reasons) and in remote storage (e.g. in an S3 bucket which is usually specifically created for it). A place where the Terraform stores state file is called the **Backend**.
 When Terraform runs, it locks the remote state so it can't be modified by other users while the infrastructure is being provisioned; this prevents state errors that can occur when two or more engineers modify the same resources at the same time.

**Init, plan, apply, destroy and refresh**

The four main terraform commands you'll use often. Let's see what each of them does.

- `terraform init` in a directory with configuration files pulls all the necessary modules to be used later and connects to Terraform state (setting it up locally or remotely);
- `terraform plan` allows to, well, plan your infrastructure changes without applying them;
- `terraform apply` runs `plan` and provisions everything described in the code after prompting for approval;
- `terraform destroy` does the opposite, planning the deletion and then deleting any resources described in the code after prompting for approval;
- `terraform refresh` reads the current settings from all managed resources and updates the Terraform state to match. It **does not** add any new resources (not described in the configuration) to the Terraform state.

**Validate and fmt**

Two simple but very handy commands to make sure your code is valid and looks good. Use them regularly. Or even better - automatically!

- `terraform validate` command validates the consistency and syntax of configuration files in a directory, referring only to the configuration and not accessing any remote services such as remote state, provider APIs, etc;
- `terraform fmt` rewrites Terraform configuration files according to [a canonical format and style](https://www.terraform.io/language/syntax/style).


![terraform apply --auto-approve](https://i.imgur.com/LRbRrAB.jpg)

**Providers**

Let's circle back to [providers](https://www.terraform.io/language/providers). These are basically Terraform subcomponents that each handle the interactions with specific platform like AWS or Azure. Providers are developed by Hashicorp itself, Terraform community and third party companies that want their services to be easily available to Terraform users.

If you're working with both AWS and Azure (or Digital Ocean, GCP etc.), you will need to specify said providers and their versions in the code. A provider allows you to address resources specific to them, e.g. the AWS provider works with resources like EC2 instances, IAM policies, security groups and many others, the Kubernetes provider works with namespaces, deployments, ingresses, configmaps and so on.

At the same time, there are basic providers that allow Terraform to work with, say, files, external binaries, security keys or just random numbers. These can all be very useful in defining your infrastructure.

It is important to note that even mature providers like AWS don't have some resources offered by their corresponding native services. Just take at look at its [changelog](https://github.com/hashicorp/terraform-provider-aws/blob/main/CHANGELOG.md). So, while Terraform is indeed powerful, it sometimes can't do everything, as it depends on providers making their resources addressable.


![interconnected](https://i.imgur.com/VGx7ETu.jpg)

**Data sources and outputs**

Terraform [data sources](https://www.terraform.io/language/data-sources) encompass various queries that Terraform can execute to get the necessary information from outside of Terraform itself, another configuration or some function. For example, your application can be provisioned using several Terraform states which need to share some data like VPC or security group ids. If said ids change, running `terraform apply` will provide their new values in real time.

So, how a data source gets some data from another Terraform state? That state needs to have defined [outputs](https://www.terraform.io/language/values/outputs). These can have any useful information like RDS hostnames, CIDRs or security group ids written to them. Outputs are also used in modules.

**Modules**

Terraform [modules](https://www.terraform.io/language/modules) are basically classes. They can make your code reusable in multiple configurations. Need to provision EKS clusters for Application A and Application B? Use [the official module](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest)! Need more features or specific customizations? Modify it or create your own module from scratch! Although the last option means you are now maintaining your own module, which is a non-trivial task.

A module usually needs you to provide the necessary variables like VPC CIDRs or Kubernetes cluster version. It then uses its own configuration with predefined providers and resources to quickly provision the specified infrastructure piece, while also providing useful outputs.

Modules are, again, [plentiful](https://registry.terraform.io/) and are developed by a massive worldwide community.

**Variables and locals**

Think of Terraform [variables](https://www.terraform.io/language/values/variables) as function arguments. They let you customize aspects of Terraform modules without altering the module's own source code. This allows you to share modules across different Terraform configurations, making your module composable and reusable. The aforementioned outputs are like function return values.

Short example on [variables precedence](examples/variables_precedence.md).

[Locals](https://www.terraform.io/language/values/locals) are simply transformed values used by a module or configuration. You can think of them as temporary local variables of a function needed for it to work properly.

**Functions**

HCL, as most other languages, provides a standard library of [functions](https://www.terraform.io/language/functions) you can use to make mutations to variables and any other data you need to pass around. Need to multiply something or split a string? There are functions for that. Need to decode some JSON data or get an md5 hash? There are functions for that as well. Generate some numbers with set step value? You got it.

**Workspaces**

Each Terraform configuration has an associated backend that defines how operations are executed and where persistent data such as the Terraform state are stored (locally, in an S3 bucket, in Consul and so on).

The persistent data stored in the backend belongs to a [workspace](https://www.terraform.io/language/state/workspaces). Initially the backend has only one workspace, called "default", and thus there is only one Terraform state associated with that configuration. Certain backends support multiple named workspaces, allowing multiple states to be associated with a single configuration. The configuration still has only one backend, but multiple distinct instances of that configuration to be deployed without configuring a new backend or changing authentication credentials.

Creating additional workspaces is optional, but can be useful in certain situations (for example, if you have multiple EKS clusters in different AWS regions).

You can also use an additional tool like [Terraspace](https://github.com/boltops-tools/terraspace) or [Terragrunt](https://github.com/gruntwork-io/terragrunt). These are also completely optional, but can make your workflow easier if you need the features they offer (and you are willing to invest additional time to learn them).

## Terraform Code Snippets

The raw theory is nothing without practical examples.

So, you can find actual code examples [there](examples/README.md#terraform-code-snippets).

## Terraform in 15 minutes

For a brief but still substantial overall explanation of what Terraform is please refer to this video:

[![Terraform in 15 minutes](https://res.cloudinary.com/marcomontalbano/image/upload/v1642606872/video_to_markdown/images/youtube--l5k1ai_GBDE-c05b58ac6eb4c4700831b2b3070cd403.jpg)](https://www.youtube.com/watch?v=l5k1ai_GBDE "Terraform in 15 minutes")


## Useful links

- [Terraform documentation](https://www.terraform.io/docs). You will use it **a lot**. Most important sections to look at first:

    - Terraform [files and directories](https://developer.hashicorp.com/terraform/language/files) documentation should help you to start with you first configuration.

    - Tearraform language [description](https://developer.hashicorp.com/terraform/language) and [syntax](https://www.terraform.io/language/syntax/configuration).

    - [style conventions](https://www.terraform.io/language/syntax/style)

    - [conditional expressions](https://www.terraform.io/language/expressions/conditionals)


- Terraform cloud related tutorials. Beginner friendly and quick:
  - [AWS](https://learn.hashicorp.com/collections/terraform/aws-get-started)
  - [GCP](https://developer.hashicorp.com/terraform/tutorials/gcp-get-started)
  - [Azure](https://developer.hashicorp.com/terraform/tutorials/azure-get-started)

- [A longer (2+ hours) Terraform tutorial video](https://www.youtube.com/watch?v=SLB_c_ayRMo). More comprehensive and detailed.

- [Stacked resource issue](./examples/stacked_resources.md)

## Check yourself

Try to answer these questions using the knowledge you've just gained. Imagine you have provisioned a configuration with an EC2 instance in AWS using `terraform apply`, so you have a) your configuration files, b) local Terraform state with said EC2 instance already in it and c) actual provisioned working resources in AWS.

1. What will happen to a), b) and c) after you delete a resource (an EC2 instance for example) **from its configuration file** and use `terraform apply`?
2. What will happen after you delete the same resource **in AWS** and use `terraform apply`?
3. What will happen after you delete the same resource **in your Terraform state** and use `terraform apply`?
4. What will happen after you change the same resource **in the configuration file** and use `terraform apply`?
5. What will happen after you delete the same resource **from its configuration file** and use `terraform destroy`?

And some additional questions to think about:

1. What's the difference between a provider and a module?
2. What will happen if you delete you Terraform state file?
3. What will happen if you delete a provider from your local `.terraform` directory in your configuration?
4. What will happen if you delete the entire `.terraform` directory in your configuration?
5. What will happen if you import existing resource with the following command: `terraform import aws_s3_bucket.my_super_storage {bucket-name}`? Do you need to do anything else to manage this resource?
6. You've created IAM user and credentials using `aws_iam_access_key` resource. After some time you'd like to import that user to another state and find that according to [documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_access_key) secret cannot be imported. What should you do then?

If you're certain you have answered all (or most) of them correctly, great job! You're most definitely ready for the main part of this lab.


## Terraform best practices

  - [A comprehensive compilation of best practices by Anton Babenko and Terraform community](https://www.terraform-best-practices.com/)

  - [A few more best practices and useful hints](https://github.com/ozbillwang/terraform-best-practices)

  - [Workflow recommended practices by Hashicorp](https://www.terraform.io/cloud-docs/guides/recommended-practices)


## Certification and Trainings

- [HashiCorp Certified: Terraform Associate](https://www.hashicorp.com/certification/terraform-associate) 
