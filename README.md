# AWS Training - Terraform Hands-on
Here you can find more details regarding hands-ons presented in the Terraform training module.

# Preparation
1. Fork training repository to your account
    * https://github.com/Alegres/awstraining-terraform

2. Install Terraform from PowerShell ​

```
choco install terraform --version=1.7.0 –force​
```

3. Install AWS CLI from PowerShell ​

```
choco install awscli​
```

4. Create profile in AWS credentials

To be able to apply Terraform changes locally, you should create a new profile in ```C:\Users\YOURUSER\.aws\credentials``` and set credentials to your account.
```
[backend-test]​
aws_access_key_id = YOUR_ACCESS_KEY_ID​
aws_secret_access_key = YOU_SECRET_ACCESS_KEY​
```

**DO NOT USE ROOT USER CREDENTIALS! ​**
Instead, create admin user in IAM, assign him AdministratorAccess policy and generate credentials for this non-root user.​

# 01 Create S3 bucket
Documentation:
* https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket
* https://registry.terraform.io/providers/hashicorp/aws/latest/docs#provider-configuration
* https://developer.hashicorp.com/terraform/language/modules/sources#local-paths


1. Go to given file ```aws-infrastructure/terraform/modules/bucket/main.tf``` and implement module

```hcl
resource "aws_s3_bucket" "bucket" {
  bucket = var.name
}
```

2. Go to given file ```aws-infrastructure/terraform/modules/bucket/vars.tf``` and implement empty module variables
   
```hcl
variable "name" {}
```

3. Go to given file ```aws-infrastructure/terraform/common/general/bucket/main.tf``` and implement main resource
   
```hcl
provider "aws" {
  region                  = var.region
  shared_credentials_files = [ var.shared_credentials_file ]
  profile                 = var.profile
}

module "bucket" {
  source = "../../../modules/bucket"
  name = "<<UNIQUE_BUCKET_NAME>>"
}
```

4. Go to given file ```aws-infrastructure/terraform/common/general/bucket/vars.tf``` and implement resource variables

```hcl
variable "region" {
  description = "Region to launch configuration in"
}

variable "profile" {
  description = "Default profile id"
}

variable "shared_credentials_file" {
  description = "Path to cloud credentials"
}

variable "environment" {}
```

5. Go to given file aws-infrastructure/terraform/common/general/bucket/versions.tf and implement Terraform versions

```hcl
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "~> 5.0.1"
    }
  }
  required_version = ">= 1.4.6"
}
```

6. Initiate Terraform for the created resource

Go to the bucket directory:
```
cd aws-infrastructure/terraform/common/general/bucket/
```

Initiate Terraform:
```
terraform init
```

This will install all modules required by this configuration

7. Start creation of the AWS infrastructure
   
```hcl
terraform apply \
  -var='environment=emea' \
  -var='profile=backend-test' \
  -var='region=eu-central-1' \
  -var='shared_credentials_file=C:\\Users\\<<USERNAME>>\\.aws\\credentials'
```

8. Check if S3 bucket has been createt in AWS
9. Destroy of AWS infrastructure

```hcl
terraform destroy \
  -var='environment=emea' \
  -var='profile=backend-test' \
  -var='region=eu-central-1' \
  -var='shared_credentials_file=C:\\Users\\<<USERNAME>>\\.aws\\credentials'
```
