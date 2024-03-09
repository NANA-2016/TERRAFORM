# TERRAFORM

Create a folder named PBL and a file named main.tf which is a file used for provider 

configurations eg AWS which is our provider in this case

We also need to  download plugins for provisioners which is done by terraform after we run the

command "terraform init" 

## HARD CORDING (though a bad practise)

##### CREATE VPC in main.tf as a resource

###### see code below 

provider "aws" {
  region = "eu-central-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}

### terra form commands .

We can now run the command 

"terraform plan" so as to see what terraform is planning to do and

late we run the command 

"terraform apply"so that chanes can be excecuted and they will be reflected in AWS.

"terraform destroy" is a coomand used by terraform to undo what changes already made and 

reflected in AWS.

###### terraform.tfstate 

Is a teraform file created after runniing terraform init conmmand and it enable terraform to

be keep itself upto date with the exact state of infrastructure based on the entire terraform 

code 

###### terraform.tfstate.lock.info

Is a file that is also created with terraform init coomand but gets deleted automatically and 

immediately . It helps terraform track who is running code against infrastructure in a team 

hence avoiding duplication and conflict

### CREATE SUBNETS

###### see code below 

# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-central-1a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-central-1b"
}

Run terraform plan then terraform apply to apply the changes . Thse changes can be see on AWS

console 

![Screenshot 2024-03-08 130041](https://github.com/NANA-2016/TERRAFORM/assets/141503408/5c6514fd-b31c-4a90-89c3-22a59ef70291)


![Screenshot 2024-03-08 125859](https://github.com/NANA-2016/TERRAFORM/assets/141503408/49f45cec-e211-4861-9729-f5e026d17ee3)

## INTRODUCTION OF VARIABLES(To stop hard coding )

#### Code refactoring 

The code below shows variable for region,cidr and vpc have bben declared 

region-"eu-central-1"

cidr -default = "172.16.0.0/16"

vpc - "aws_vpc"

###### see code below 

    variable "region" {
        default = "eu-central-1"
    }

    provider "aws" {
        region = var.region
    }
    variable "region" {
        default = "eu-central-1"
    }

    variable "vpc_cidr" {
        default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
        default = "true"
    }

    variable "enable_dns_hostnames" {
        default ="true" 
    }

    variable "enable_classiclink" {
        default = "false"
    }

    variable "enable_classiclink_dns_support" {
        default = "false"
    }

    provider "aws" {
    region = var.region
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support 
    enable_dns_hostnames           = var.enable_dns_support
    enable_classiclink             = var.enable_classiclink
    enable_classiclink_dns_support = var.enable_classiclink

    }
### REMOVING HARD CORDED COUNT VALUE

Here the concept of length()is introduced  which determineds the number of a given list,map or 

string .
an exapmple as given in the code below "length(data.aws_availability_zones.available.names)"

#### Introducing the count variable 

count  = var.preferred_number_of_public_subnets == null ?(as see on the last part of the code )

This helps us create the right number of the required subnets


###### see code below

# Get list of availability zones
data "aws_availability_zones" "available" {
state = "available"
}

variable "region" {
      default = "eu-central-1"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

variable "enable_classiclink" {
    default = "false"
}

variable "enable_classiclink_dns_support" {
    default = "false"
}

  variable "preferred_number_of_public_subnets" {
      default = 2
}

provider "aws" {
  region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support 
  enable_dns_hostnames           = var.enable_dns_support
  enable_classiclink             = var.enable_classiclink
  enable_classiclink_dns_support = var.enable_classiclink

}


# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}

to see how it works, we open terraform console and run the command for example 
####### length(["eu-central-1a", "eu-central-1b", "eu-central-1c"])

## VARIABLES AND tfvars files 
We create files to be able to separate the code and make it easy to read using the files

listed 

below with there sample codes.

###### main .tf

###### see code below

# Get list of availability zones
data "aws_availability_zones" "available" {
state = "available"
}

provider "aws" {
  region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support 
  enable_dns_hostnames           = var.enable_dns_support
  enable_classiclink             = var.enable_classiclink
  enable_classiclink_dns_support = var.enable_classiclink

}

# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
}


###### variable.tf file 
 In this file , all variables are declared here .
 ###### See code below
 
variable "region" {
      default = "eu-central-1"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

variable "enable_classiclink" {
    default = "false"
}

variable "enable_classiclink_dns_support" {
    default = "false"
}

  variable "preferred_number_of_public_subnets" {
      default = null
}
##### teraform.tfvars

This file sets variables for each of the set variables in the variable.tf file 

###### see code below

region = "eu-central-1"

vpc_cidr = "172.16.0.0/16" 

enable_dns_support = "true" 

enable_dns_hostnames = "true"  

enable_classiclink = "false" 

enable_classiclink_dns_support = "false" 

preferred_number_of_public_subnets = 2

![image](https://github.com/NANA-2016/TERRAFORM/assets/141503408/e5110095-2645-489d-9aad-be40927ea392)


![image](https://github.com/NANA-2016/TERRAFORM/assets/141503408/aa590c26-1ea4-4265-a8b3-957eb96140ea)


### PBL FOLDER END RESULTS

![image](https://github.com/NANA-2016/TERRAFORM/assets/141503408/0ce87a35-df4b-4e22-a8d5-eb620b452361)
