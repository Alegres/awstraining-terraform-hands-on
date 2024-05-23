# AWS Training - Terraform Hands-on
## Create bucket
Documentation:
* https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket
* https://registry.terraform.io/providers/hashicorp/aws/latest/docs#provider-configuration
* https://developer.hashicorp.com/terraform/language/modules/sources#local-paths


1. Go to given file ```aws-infrastructure/terraform/modules/bucket/main.tf``` and implement:
```hcl
resource "aws_s3_bucket" "bucket" {
  bucket = var.name
}
```
2. Go to given file aws-infrastructure/terraform/modules/bucket/vars.tf and implement:
```hcl
variable "name" {}
```
3. Go to given file aws-infrastructure/terraform/common/general/bucket/main.tf and implement:
```hcl
provider "aws" {
  region                  = var.region
  shared_credentials_files = [ var.shared_credentials_file ]
  profile                 = var.profile
}

module "bucket" {
  source = "../../../modules/bucket"
  name = "<<UNIQ_BUCKET_NAME>>"
}
```
4. Go to given file aws-infrastructure/terraform/common/general/bucket/vars.tf and implement:
```hcl
variable "region" {
  description = "Region to launch configuration in"
}
variable "profile" {
  description = "Default profile id"
}
variable "name" {
  description = "Name of the bucket"
}

variable "shared_credentials_file" {
  description = "Path to cloud credentials"
}

variable "environment" {}
```
5. Go to given file aws-infrastructure/terraform/common/general/bucket/versions.tf and implement:
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
5. Go to given directory
   cd aws-infrastructure/terraform/common/general/create-remote-state-bucket/
6. Initiate terraform -> terraform init  This will install all modules required by this configuration. 
7. Start creation of AWS infrastructure ->
```hcl
terraform apply \
  -var='environment=emea' \
  -var='profile=backend-test' \
  -var='region=eu-central-1' \
  -var='shared_credentials_file=C:\\Users\\<<USERNAME>>\\.aws\\credentials'

```
8. Check if S3 bucket has been createt in AWS
9. Destroy of AWS infrastructure ->
```hcl
terraform destroy \
  -var='environment=emea' \
  -var='profile=backend-test' \
  -var='region=eu-central-1' \
  -var='shared_credentials_file=C:\\Users\\<<USERNAME>>\\.aws\\credentials'
