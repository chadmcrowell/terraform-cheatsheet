# terraform-cheatsheet

[command docs](https://www.terraform.io/docs/commands/)
[functions](https://www.terraform.io/docs/configuration/functions/)
[provisioners](https://www.terraform.io/docs/provisioners/index.html)
[modules](https://registry.terraform.io/browse/modules)

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

## Terraform Workspace
* use different state files for each workspace
```bash
tf workspace show       # show current workspace

tf workspace list       # list all available workspaces

tf workspace new dev        # create and switch to new workspace named dev

tf workspace new prod       # create a new workspace named prod

tf workspace select dev     # switch to the dev workspace
```

### Workspace - Example
```hcl
resource "aws_instance" "myec2" {
   ami = "ami-082b5a644766e0e6f"
   instance_type = lookup(var.instance_type,terraform.workspace)
}

variable "instance_type" {
  type = "map"

  default = {
    default = "t2.nano"
    dev     = "t2.micro"
    prd     = "t2.large"
  }
}
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
tf init      # initialize terraform config files

tf init -upgrade        # upgrade to the latest provider 
```

## Envrionment Variable
```bash
setx TF_VAR_instancetype m5.large       # set a variable via environment variable (Windows)

export TF_VAR_instancetype="t2.nano"        # set a variable via environment variable (Linux)
```

## Terraform Get
```bash
tf get      # install and update modules
```

## Terraform Plan
```bash
tf plan     # pre-deployment plan

tf plan -out=newplan    # save the plan to a file

tf plan -var="instancetype=t2.small"        # explicitly define variable value

tf plan -var-file="custom.tfvars"       # use custom tfvars file name

tf plan -target aws_security_group.sg_allow_ssh     # detect changes in a specific resource

tf plan -refresh=false      # skip the refresh of all resources in the configuration

tf plan -destroy        # plan a destroy without committing
```

## Terraform Apply
```bash
tf apply    # prompt before creating resources described in tf

tf apply --auto-approve     # create resources without prompt

tf apply "newplan"      # apply plan from plan file
```

## Terraform Destroy
```bash
tf destroy      # destroy all

tf destroy -target aws_instace.my_instance      # only destroy specific resource
```

## Terraform Taint
```bash
tf taint aws_instance.myec2     # mark ec2 instance to destroy and recreate on next apply
```

## Terraform State
```bash
tf state show aws_eip.myeip    # inspect the state of an elastic ip resource

tf state list   # list the resources in the state file

tf state refresh      # refresh the current state

tf state mv aws_instance.my_webapp aws_instance.my_ec2      # move ec2 instance within state without destroying and recreating

tf state pull     # manually download and output the remote state

tf state push     # manually upload a local state file to remote loc

tf state rm aws_instance.my_ec2      # removes ec2 instance from state (tf no longer aware)
```

## Terraform Graph
```bash
tf graph    # visual dependency graph
```

## Terraform Import
```bash
tf import azurerm_storage.account.storage_account       # import the azure storage account specified in tf

tf import aws_instance.myec2 i-041886ebb7e97bd20        # import ec2 instance with instance ID

tf import module.vm.azurerm_linux_virtual_machine vm1   # import azure vm named "vm1" into module "vm"
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

## Count and Count Index
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

## Multiple Resources with Multiple Regions/Accounts
```hcl
# main.tf

# use alias for multiple providers blocks
provider "aws" {
  region     = "us-east-1"
  version    = ">=2.8"
}

provider "aws" {
  alias      =  "mumbai"
  region     =  "ap-south-1"
}

resource "aws_eip" "myeip" {
  vpc = "true"
}

resource "aws_eip" "myeip01" {
  vpc = "true"
  provider = aws.mumbai
}
```
```hcl
# main.tf

# use different creds from ~/.aws/credentials
provider "aws" {
  region     = "us-east-1"
  version    = ">=2.8"
}

provider "aws" {
  alias      = "tfadmin2cred"
  region     = "ap-south-1"
  profile    = "tfadmin2"
}

resource "aws_eip" "myeip" {
  vpc = "true"
}

resource "aws_eip" "myeip01" {
  vpc = "true"
  provider = aws.tfadmin2cred
}
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
}

data "aws_ami" "my_ami" {
  most_recent = true
  owners = ["amazon"]


  filter {
    name   = "name"
    values = ["amzn2-ami-hvm*"]
  }
}

resource "aws_instance" "instance-1" {
    ami = data.aws_ami.my_ami.id
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

## Terraform Modules
```hcl
# myec2.tf
module "ec2module" {
    source = "./modules/ec2"
}

# ./modules/ec2/ec2.tf
resource "aws_instance" "myec2" {
    ami = "ami-0b1e534a4ff9019e0"
    instance_type = "t2.micro"
}
```
```hcl
# myec2.tf
module "ec2module" {
    source = "./modules/ec2"
    instance_type = "m4.large"
}

# ./modules/ec2/variables.tf
variable "instance_type" {
    default = "t2.micro"
}

# ./modules/ec2/ec2.tf
resource "aws_instance" "myec2" {
    ami = "ami-0b1e534a4ff9019e0"
    instance_type = var.instance_type
}
```

## Module Sources
```hcl
# main.tf

# module source is a generic git repo
module "my_module" {
    source = "git::https://github.com/chadmcrowell/tmp-repo.git"
}
```
```hcl
# main.tf

# module source is a github repo
module "my_module" {
    source = "github.com/chadmcrowell/tmp-repo"
}
```
```hcl
# main.tf

# module source is a specific branch of a generaic github repo
module "my_module" {
    source = "git::https://github.com/chadmcrowell/tmp-repo.git?ref=develop"
}
```

## Registry Module
```hcl
module "ec2_cluster" {
  source                 = "terraform-aws-modules/ec2-instance/aws"
  version                = "~> 2.0"

  name                   = "my-cluster"
  instance_count         = 1

  ami                    = "ami-0d6621c01e8c2de2c"
  instance_type          = "t2.micro"
  subnet_id              = "subnet-4dbfb206"

  tags = {
    Terraform   = "true"
    Environment = "dev"
  }
}
```

## Terraform State - AWS Backend
```hcl
# backend.tf

# store the terraform.tfstate file in an s3 bucket
terraform {
    backend "s3" {
        bucket = "my-remote-state-s3-bucket"
        key    = "terraform.tfstate"
        region = "us-east-1"
        access_key = "MY_ACCESS_KEY"
        secret_key = "MY_SECRET_KEY"
    }
}
```
```hcl
# backend.tf

# store the terraform.tfstate file in an s3 bucket with STATE LOCK enabled
terraform {
    backend "s3" {
        bucket         = "my-remote-state-s3-bucket"
        key            = "terraform.tfstate"
        region         = "us-east-1"
        access_key     = "MY_ACCESS_KEY"
        secret_key     = "MY_SECRET_KEY"
        dynamodb_table = "s3-state-lock"
    }
}
```

[backend types](https://www.terraform.io/docs/backends/types/)

## Assume Role in AWS
```hcl
provider "aws" {
    region = "us-west-1"
    assume_role {
        role_arn = "arn:aws:iam::871285060102:role/test-sts"
        session_name = "sts-demo"
    }
}
```

## Sensitive Parameter
```hcl
locals {
    db_password = {
        admin = "password"
    }
}

output "db_password" {
    value     = local.db_password
    sensitive = true
}
```

## Sentinel Policy
[sentinel - docs](https://docs.hashicorp.com/sentinel/terraform/)
```hcl
# policy in terraform cloud

import "tfplan"
 
main = rule {
  all tfplan.resources.aws_instance as _, instances {
    all instances as _, r {
      (length(r.applied.tags) else 0) > 0
    }
  }
}
```

## Remote Backend
* Apply not allowed for workspaces with a VCS connection
```hcl
# main.tf
terraform {
    required_version = "~> 0.12.0"

    backend "remote" {}
}

resource "aws_iam_user" "ec2user" {
    name = "ec2user"
    path = "/system"
}
```
```hcl
# backend.hcl

workspaces { name = "my-repository-name" }
hostname     = "app.terraform.io"
organization = "my-organization-name"
```


## 3rd Party Providers
Operating System | User plugins directory  
------- | ----------------  
Windows | %APPDATA%\terraform.d\plugins  
All other OS | ~/.terraform.d/plugins  

## .gitignore
Files to Ignore | Description
------- | ----------------  
`.terraform` | This file created when `tf init` is run  
`terraform.tfvars` | contains all of your secrets
`terraform.tfstate` | should be stored remotely
`crash.log` | logs are stored here if tf crashes
