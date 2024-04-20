# Stacked Resources Issue

## Explanation

Sometimes there are several ways to create the same resources.
One of the most typical cases is an EC2 instance creation.

Imagine that we have a terraform-managed EC2 instance:
```
data "aws_ami" "ubuntu_ami" {
  most_recent = true
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  owners = ["099720109477"]
}

resource "aws_instance" "lab_instance" {
  ami           = data.aws_ami.ubuntu_ami.id
  availability_zone = "us-east-1b"
  instance_type = "t3.micro"

  root_block_device {
    volume_size    = 8
    volume_type    = "gp2"
  }
}
```

Now we need to add an EBS volume to this instance.
We can do it in several ways.

### Option 1 (Nested or stacked resource)
```
resource "aws_instance" "lab_instance" {
  ami           = data.aws_ami.ubuntu_ami.id
  availability_zone = "us-east-1b"
  instance_type = "t3.micro"

  root_block_device {
    volume_size    = 8
    volume_type    = "gp2"
  }
  ebs_block_device {
    device_name    = "/dev/sdb"
    volume_size    = 8
    volume_type    = "gp2"
  }
}
```

In that case, when we run terraform plan we will see the next:
```bash
> terraform plan
aws_instance.lab_instance: Refreshing state... [id=i-0d702c2c782a2d28f]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # aws_instance.lab_instance must be replaced
-/+ resource "aws_instance" "lab_instance" {
    <...>
    }

Plan: 1 to add, 0 to change, 1 to destroy.
```

`-/+` means that the resource will be removed and then created. In other words, downtime is expected.

### Option 2 (Separated resources)
```
resource "aws_instance" "lab_instance" {
  ami           = data.aws_ami.ubuntu_ami.id
  availability_zone = "us-east-1b"
  instance_type = "t3.micro"

  root_block_device {
    volume_size    = 8
    volume_type    = "gp2"
  }
}

resource "aws_ebs_volume" "lab_ebs" {
  availability_zone ="us-east-1b"
  size              = 8
}

resource "aws_volume_attachment" "lab_ebs_attachement" {
  device_name = "/dev/sdb"
  volume_id   = aws_ebs_volume.lab_ebs.id
  instance_id = aws_instance.lab_instance.id
}
```

In that case, when we run terraform plan we will see the next:
```bash
❯ terraform plan
aws_instance.lab_instance: Refreshing state... [id=i-0e637595b37d3420a]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_ebs_volume.lab_ebs will be created
  + resource "aws_ebs_volume" "lab_ebs" {
    <...>
    }

  # aws_volume_attachment.lab_ebs_attachement will be created
  + resource "aws_volume_attachment" "lab_ebs_attachement" {
    <...>
    }

Plan: 2 to add, 0 to change, 0 to destroy.
```

In this example, the instance will not be recreated. Only new resources will be attached without availability impact.

**Two options. First with potential downtime, second - without.**

Let's consider other examples.

## Examples
### AWS Security Group with rules
#### Stacked Resources Option

Security groups could be created with  included rules (`ingress` and `egress` blocks). Pros is that you can easy read the security group and understand what is managed by it. From the other side any change to any rule, like even change description for ingress rule will lead to destruction of whole security group, what means downtime again.

Another disadvantage could be faced with looping reference to security group ids. If you have `security_group_A` which allows only trafic from `security_group_B` only and vice versa. You cannot create such groups in case if you define those rules inside the group, it leads to a loop while one security group waiting for another to be created. Having security group rules as standalone resources allows securtity groups to be created it this case.

```
resource "aws_security_group" "lab_sg" {
  name        = "allow_internal_tls"
  description = "Allow TLS inbound traffic"
  vpc_id      = "vpc-h542xt2g"

  ingress {
    description      = "TLS from VPC"
    from_port        = 443
    to_port          = 443
    protocol         = "tcp"
    cidr_blocks      = ["172.31.69.0/24"]
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
  }
}
```

#### Separated Resources Option
```
resource "aws_security_group" "lab_sg" {
  name        = "allow_internal_tls"
  description = "Allow TLS inbound traffic"
  vpc_id      = "vpc-h542xt2g"
}

resource "aws_security_group_rule" "ingress_tls_from_vpc" {
  type              = "ingress"
  from_port         = 443
  to_port           = 443
  protocol          = "tcp"
  description       = "TLS from VPC"
  cidr_blocks       = ["172.31.69.0/24"]
  security_group_id =  aws_security_group.lab_sg.id
}

resource "aws_security_group_rule" "egress_to_the_internet" {
  type              = "egress"
  from_port         = 0
  to_port           = 0
  protocol          = "-1"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id =  aws_security_group.lab_sg.id
}
```
### AWS IAM Role with managed policy
#### Stacked Resources Option

IAM Role can recieve policy with managed_policy_arns parameter. Or we can use `aws_iam_role_policy_attachment` resource. In first case you cannot change or add policy without resource recreation.

```
resource "aws_iam_role" "example_role" {
  name = "example-role"
  assume_role_policy = data.aws_iam_policy_document.assume_role_policy.json
  managed_policy_arns = [
    "arn:aws:iam::aws:policy/AdministratorAccess",
  ]
}
```

#### Separated Resources Option
```
resource "aws_iam_role" "example_role" {
  name = "example-role"
  assume_role_policy = data.aws_iam_policy_document.assume_role_policy.json
}

resource "aws_iam_role_policy_attachment" "example_role_admin" {
  role       = aws_iam_role.example_role.name
  policy_arn = "arn:aws:iam::aws:policy/AdministratorAccess"
}
```


### Azure Network Security Group
#### Stacked Resources Option
```
resource "azurerm_resource_group" "lab_resource_group" {
  name     = "lab-resources"
  location = "West Europe"
}

resource "azurerm_network_security_group" "lab_nsg" {
  name                = "labNSG"
  location            = azurerm_resource_group.lab_resource_group.location
  resource_group_name = azurerm_resource_group.lab_resource_group.name

  security_rule {
    name                       = "AllowAllTcp"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "*"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}
```

#### Separated Resources Option
```
resource "azurerm_resource_group" "lab_resource_group" {
  name     = "lab-resources"
  location = "West Europe"
}

resource "azurerm_network_security_group" "lab_nsg" {
  name                = "labNSG"
  location            = azurerm_resource_group.lab_resource_group.location
  resource_group_name = azurerm_resource_group.lab_resource_group.name
}

resource "azurerm_network_security_rule" "example" {
  name                        = "AllowAllTcp"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "*"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.lab_resource_group.name
  network_security_group_name = azurerm_network_security_group.lab_resource_group.name
}
```
