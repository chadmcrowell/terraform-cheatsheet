# terraform-cheatsheet

[command docs](https://www.terraform.io/docs/commands/)

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
```

## Terraform Apply
```bash
tf apply    # prompt before creating resources described in tf

tf apply --auto-approve     # create resources without prompt

tf apply "newplan"      # apply plan from plan file
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

## 3rd Party Providers
Operating System | User plugins directory  
------- | ----------------  
Windows | %APPDATA%\terraform.d\plugins  
All other OS | ~/.terraform.d/plugins  