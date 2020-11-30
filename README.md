# terraform-cheatsheet

[command docs](https://www.terraform.io/docs/commands/)
[functions](https://www.terraform.io/docs/configuration/functions/)
[provisioners](https://www.terraform.io/docs/provisioners/index.html)

Version Number Arguments | Description  
------- | ----------------  
\>=2.0 | Greater than or equal to version 2.0  
<=3.0 | Less than or equal to version 3.0  
~>3.0 | Any version in the 3.0 range  
\>=3.10,<=3.18 | Any version between 3.10 and 3.18  

## Alias
```bash
alias tf=terraform
```

## Terraform Console
```bash
tf console      # access interactive command-line console to experiment with expressions
```

## Terraform Validate
```bash
tf validate     # validate the configuration for syntax validity
```

## Terraform Format
```bash
tf fmt      # format the configuration files to standardize style
```

## Terraform Upgrade
```bash
tf 0.13upgrade .        # upgrade to major release v0.13
```

## Terraform Init
```bash
terraform init      # initialize terraform config files
```

## Envrionment Variable
```bash
setx TF_VAR_instancetype m5.large       # set a variable via environment variable (Windows)

export TF_VAR_instancetype="t2.nano"        # set a variable via environment variable (Linux)
```

## Terraform Plan
```bash
tf plan     # pre-deployment plan

tf plan -out=newplan    # save the plan to a file

tf plan -var="instancetype=t2.small"        # explicitly define variable value

tf plan -var-file="custom.tfvars"       # use custom tfvars file name

tf plan -target aws_security_group.sg_allow_ssh     # detect changes in a specific resource

tf plan -refresh=false      # skip the refresh of all resources in the configuration
```

## Terraform Apply
```bash
tf apply    # prompt before creating resources described in tf

tf apply --auto-approve     # create resources without prompt

tf apply "newplan"      # apply plan from plan file
```

## Terraform Taint
```bash
tf taint aws_instance.myec2     # destroy and recreate the resource aws_instance.myec2
```

## Terraform State
```bash
tf show     # inspect the current state

tf state list   # list the resources in the state file

tf refresh      # refresh the current state
```

## Terraform Graph
```bash
tf graph    # visual dependency graph
```

## Terraform Import
```bash
tf import azurerm_storage.account.storage_account       # import the azure storage account specified in tf
```

## Terraform Output
```bash
tf output iam_arm       # output the value iam_arn specified within the output configuration
```

## HEREDOC
```hcl
# input stream literal and redirect to command
# insert script from source for azure vm extension

extension {
    name                 = "${var.server_name}-extension"
    publisher            = "Microsoft.Compute"
    type                 = "CustomScriptExtension"
    type_handler_version = "1.10"

    settings = <<SETTINGS
    {
        "fileUris" : ["https://raw.githubusercontent.com/eltimmo/learning/master/azureInstallWebServer.ps1"],
        "commandToExecute" : "start powershell -ExecutionPolicy Unrestricted -File azureInstallWebServer.ps1"
    }
    SETTINGS
  }
```

## Count
```hcl
/* 
if the environment variable is equal to "prod"
then create 0(none), else create 1
*/
count = var.environment == "production" ? 0 : 1

/*
if the environment variable is equal to "prod"
then set allocation method to "static", else set to "Dynamic"
*/
allocation_method = var.environment == "prod" ? "static" : "Dynamic"
```

## Terraform Destroy
```bash
tf destroy      # destroy all

tf destroy -target aws_instace.my_instance      # only destroy specific resource
```

## Logging
```bash
export TF_LOG=TRACE     # enable trace logging

export TF_LOG_PATH="terraform.txt"      # set log path to output to a file
```

## Terraform Block
```hcl
# main.tf

# must use terraform v0.11 or earlier
# must use aws provider in the v2.0 range
terraform {
  required_version = "< 0.11"
  required_providers {
    aws = "~> 2.0"
  }
}

provider "aws" {
  region     = "ap-southeast-1"
  access_key = "YOUR-KEY"
  secret_key = "YOUR-KEY"
}

resource "aws_instance" "myec2" {
   ami = "ami-0b1e534a4ff9019e0"
   instance_type = "t2.micro"
}
```

## Comments
```hcl
/*
resource "digitalocean_droplet" "my_droplet" {
    image = "ubuntu-18-04-x64"
    name = "web-1"
    region = "nyc1"
    size = "s-1vcpu-1gb"
}
*/
```

## Data Types
```hcl
# variables.tf
variable "data-type-example" {
    type = string
    type = list     # e.g. ["us-east-1a", "us-west-2b"]
    type = map      # e.g. {name = "Chad", age = 34}
    type = number
}
```

## Fetch Data from Map
```hcl
# main.tf
resource "aws_instance" "myec2" {
    ami = "ami082c5a44755e0e6f"
    instance_type = var.types["us-west-2"]
}

variable "types" {
    type = map
    default = {
        us-east-1 = "t2.micro"
        us-west-2 = "t2.nano"
        ap-south-1 = "t2.small"
    }
}
```

## Fetch Data from List
```hcl
# main.tf
resource "aws_instance" "myec2" {
    ami = "ami082c5a44755e0e6f"
    instance_type = var.types[0]
}

variable "types" {
    type = list
    default = ["m5.large","m5.xlarge","t2.medium"]
}
```

## Count and Count Index
```hcl
# main.tf

# create 5 ec2 instances
resource "aws_instance" "instance_one" {
    ami = "ami082c5a44755e0e6f"
    instance_type = "t2.micro"
    count = 5
}
```
```hcl
# main.tf

# create 5 users, appending the count index (0-4) for each
resource "aws_iam_user" "iam_user" {
    name = "ec2user.${count.index}"
    count = 5
    path = "/system/"
}
```
```hcl
# main.tf

# iterate through the index of variable names
variable "ec2_names" {
    type = list
    default = ["dev-ec2user", "stage-ec2user", "prod-ec2user"]
}

resource "aws_iam_user" "iam_user" {
    name = var.ec2_names[count.index]
    count = 3
    path = "/system/"
}
```
```hcl
# main.tf

# create a dev ec2 instance or prod ec2 instance
variable "dev_env" {}

resource "aws_instance" "instance_one" {
    ami = "ami082c5a44755e0e6f"
    instance_type = "t2.micro"
    count = var.dev_env == true ? 1 : 0
}

resource "aws_instance" "instance_one" {
    ami = "ami082c5a44755e0e6f"
    instance_type = "t2.large"
    count = var.dev_env == false ? 1: 0
}
```

## Locals 
```hcl
# main.tf

locals {
  common_tags = {
    Owner = "DevOps Team"
    service = "backend"
  }
}
resource "aws_instance" "app-dev" {
   ami = "ami-082b5a644766e0e6f"
   instance_type = "t2.micro"
   tags = local.common_tags
}

resource "aws_ebs_volume" "db_ebs" {
  availability_zone = "us-west-2a"
  size              = 8
  tags = local.common_tags
}
```
```hcl
# main.tf

variable "name" {}

variable "default" {}

locals {
    name_prefix = "${var.name != "" ? var.name : var.default}"
}

resource "aws_instance" "app-dev" {
   ami = "ami-082b5a644766e0e6f"
   instance_type = "t2.micro"
   name = local.name_prefix
}
```

## Functions
```hcl
# main.tf

variable "region" {
    default = "us-east-1"
}

variable "ami" {
    type = map
    default = {
        "us-east-1" = "ami-0323c3dd2da7fb37d"
        "us-west-2" = "ami-0d6621c01e8c2de2c"
        "ap-south-1" = "ami-0470e33cd681b2476"
    }
}

variable "tags" {
    type = list
    default = ["firstec2","secondec2"]
}

# use the file function to use the public key from id_rsa.pub in the module path
resource "aws_key_pair" "loginkey" {
  key_name   = "login-key"
  public_key = file("${path.module}/id_rsa.pub")
}

# use the lookup function to insert ami ID
# use the element function to interate through tags for each instance count
resource "aws_instance" "app-dev" {
   ami = lookup(var.ami,var.region)
   instance_type = "t2.micro"
   key_name = aws_key_pair.loginkey.key_name
   count = 2

   tags = {
     Name = element(var.tags,count.index)
   }
}
```

## Data Sources
```hcl
# main.tf

# retrieve the correct ami for the ap-southeast-1 region
provider "aws" {
  region     = "ap-southeast-1"
  access_key = "YOUR-ACCESS-KEY"
  secret_key = "YOUR-SECRET-KEY"
}

data "aws_ami" "app_ami" {
  most_recent = true
  owners = ["amazon"]


  filter {
    name   = "name"
    values = ["amzn2-ami-hvm*"]
  }
}

resource "aws_instance" "instance-1" {
    ami = data.aws_ami.app_ami.id
   instance_type = "t2.micro"
}
```

## Dynamic Block
```hcl
# main.tf

# dynamically create multiple security group rules for each port
# the iterator reassigns the element name
variable "sg_ports" {
  type        = list(number)
  description = "list of ingress ports"
  default     = [8200, 8201,8300, 9200, 9500]
}

resource "aws_security_group" "dynamicsg" {
  name        = "dynamic-sg"
  description = "Ingress for Vault"

  dynamic "ingress" {
    for_each = var.sg_ports
    iterator = port
    content {
      from_port   = port.value
      to_port     = port.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  dynamic "egress" {
    for_each = var.sg_ports
    content {
      from_port   = egress.value
      to_port     = egress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

## Splat Expression
```
# main.tf

# output the ARN for all 3 users
resource "aws_iam_user" "ec2_user" {
  name = "iamuser.${count.index}"
  count = 3
  path = "/system/"
}

output "arns" {
  value = aws_iam_user.ec2_user[*].arn
}
```

## Provisioner
```hcl
# main.tf

# create ec2 instance and use remote-exec to ssh into the instance and install nginx
resource "aws_instance" "myec2" {
   ami = "ami-082b5a644766e0e6f"
   instance_type = "t2.micro"
   key_name = "kplabs-terraform"

   provisioner "remote-exec" {
     inline = [
       "sudo amazon-linux-extras install -y nginx1.12",
       "sudo systemctl start nginx"
     ]

   connection {
     type = "ssh"
     user = "ec2-user"
     private_key = file("./kplabs-terraform.pem")
     host = self.public_ip
   }
   }
}
```
```hcl
# main.tf

# create ec2 instance and use local exec to copy private ip to a file
resource "aws_instance" "myec2" {
   ami = "ami-082b5a644766e0e6f"
   instance_type = "t2.micro"

   provisioner "local-exec" {
    command = "echo ${aws_instance.myec2.private_ip} >> private_ips.txt"
  }
}
```
```hcl
# main.tf

# create ec2 instance and use remote-exec to install nano via ssh
# uninstall nano when tf destroy is run
# if nano install fails, it marks resource as tainted
resource "aws_instance" "myec2" {
   ami = "ami-0b1e534a4ff9019e0"
   instance_type = "t2.micro"
   key_name = "ec2-key"
   vpc_security_group_ids  = [aws_security_group.allow_ssh.id]

   provisioner "remote-exec" {
     inline = [
       "sudo yum -y install nano"
     ]
   }
   provisioner "remote-exec" {
       when    = destroy
       inline = [
         "sudo yum -y remove nano"
       ]
     }
   connection {
     type = "ssh"
     user = "ec2-user"
     private_key = file("./ec2-key.pem")
     host = self.public_ip
   }
}
```
```hcl
# main.tf

# create ec2 instance and use remote-exec to install nano via ssh
# uninstall nano when tf destroy is run
# if nano install fails, it will NOT mark the instance as tainted
resource "aws_instance" "myec2" {
   ami = "ami-0b1e534a4ff9019e0"
   instance_type = "t2.micro"
   key_name = "ec2-key"
   vpc_security_group_ids  = [aws_security_group.allow_ssh.id]

   provisioner "remote-exec" {
     on_failure = continue
     inline = [
       "sudo yum -y install nano"
     ]
   }
   provisioner "remote-exec" {
       when    = destroy
       inline = [
         "sudo yum -y remove nano"
       ]
     }
   connection {
     type = "ssh"
     user = "ec2-user"
     private_key = file("./ec2-key.pem")
     host = self.public_ip
   }
}
```

## 3rd Party Providers
Operating System | User plugins directory  
------- | ----------------  
Windows | %APPDATA%\terraform.d\plugins  
All other OS | ~/.terraform.d/plugins  