# Terraform-Map-variable

MAP Variable in Terraform
-------------------------

For different regions, the AMI ID's also will be different.

Now we need to launch instances in 2 different regions, for this we need to use MAP variable.

us-east-1 --> different AMI ID
us-west-1 --> different AMI ID

provider.tf
-----------
provider "aws" {
  region     = var.region
  version = "~> 4.0"
  access_key = "AKIAS5QJFNKWB2YIVC22"
  secret_key = "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"
} 

aws_instance.tf
---------------
resource "aws_instance" "web" {
  ami           = lookup(var.ami-id,var.region)
  instance_type = var.instance_type
  key_name   = var.key_name 

  tags = {
    Name = var.instance_tag
  }
}

variables.tf
------------
variable "ami-id" {
    type = map
    default = {
    "us-east-1" = "ami-090fa75af13c156b4"
    "us-west-2" = "ami-0cea098ed2ac54925"
    "us-east-2" = "ami-051dfed8f67f095f5"
    "us-west-1" = "ami-0e4d9ed95865f3b40"
  }
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "key_name" {
  type    = string
  default = "8ambatch"
}

variable "instance_tag" {
  type = string
  default = "HelloWorld"
}

variable "region" {
  type = string
}

terraform apply -auto-approve
terraform destroy -auto-approve

For map variable we should use lookup, and also 2 arguments needs to be present.

If we want to pass the values from terraform command itself
-----------------------------------------------------------
terraform apply -var="region=us-east-1" -auto-approve

terraform destroy -var="region=us-east-1" -auto-approve



If we want to create 2 instances then surely we need to provide count=2 only then the second instance will be created. 


Statefile
---------
Whenever we add any resources like ec2-instance, securitygroups etc., it will be added to statefiles. if we give terraform destroy command, what are all the resources present in the statefile only those resources will be deleting. so we shouldn't place this statefile in our local, instead we need to place it in S3 bucket.

incase if the statefile is deleted mitakenly, what are all the resources we have created wont be deleting. so we need to secure this statefile so that no others will be deleting this statefile.


Add a new file named "backup.tf" in terraform under variable folder. create a bucket in amazon s3 named 8ambatch.

backup.tf
---------
terraform{
    backend "s3" {
        bucket = "8ambatch"
        encrypt = true
        key = "terraform.tfstate"
        region = "us-east-1
    }
}


before moving our statefile to s3 bucket, we need to initialize backend configuration. (one time  activity)

terraform init -backend-config="access_key=AKIAS5QJFNKWB2YIVC22" -backend-config="secret_key=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"
terraform plan
terraform apply -var="region=us-east-1" -auto-approve

Now our statefile also will be moved to s3 bucket.

So, even when anyone deletes the statefile from our local also no need to worry, we can just download this statefile from the s3 bucket and we can use it.






Output variables
----------------
These output variables we will be using to get the values like public ip, private ip etc.,

Create one more file named "output.tf".



output.tf
---------
output "ip" {
  value = aws_instance.web.*.public_ip
}


terraform init
terraform plan
terraform apply -var="region=us-east-1" -auto-approve


Outputs:

ip = [
  "54.226.47.141",
]


From now onwards,  all the resources will be stored in the s3 bucket itself rather than in our local. even if we give terraform destroy command, all the resources present in the statefile in the s3 bucket will be destroyed.

S3 bucket, IAM are global specific.



Lets say we have 2 different resources, ec2-instance and security groups.

I want to delete only 1 resource then i can give the command like

terraform destroy -target=aws_instance.web
