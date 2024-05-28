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

# Deploy application with manual commands
If you do not have Docker, Maven and JAVA installed locally, then please setup Cloud9 environment in AWS, following instructions from:
* https://github.com/Alegres/awstraining-basics-hands-on?tab=readme-ov-file#deploy-our-application

1. Go to main directory of your forked repository and build application using Maven

```bash
mvn clean install
```

2. Create Docker image

```bash
docker build -t backend .
```

3. Go to AWS -> ECR -> backend repository and click on "View push commands"

4. Login to AWS ECR by using the **ecr get-login-password** command from the pop-up dialog

```bash
aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 467331071075.dkr.ecr.eu-central-1.amazonaws.com
```

This will allow you to push Docker images.

**When running locally - it is important to provide --profile option and specify your AWS profile.**

5. Tag Docker image with the **latest** tag

```bash
docker tag backend:latest 467331071075.dkr.ecr.eu-central-1.amazonaws.com/backend:latest
```

6. Push image to ECR

```bash
docker push 467331071075.dkr.ecr.eu-central-1.amazonaws.com/backend:latest
```

7. Go to AWS -> ECR -> backend and confirm that the image was pushed

8. Go to AWS -> ECS -> Your Fargate cluster -> select your service and click on "Update"

9. Select "Force new deployment", specify "Desired tasks" to 3, leave other options untouched and click on "Update" button

10. Verify if the deployment has started

11. Under "Tasks" tab confirm that all 3 tasks are in state "Running"

12. Wait some time (around 3 minutes) and then go to AWS -> EC2 -> Target groups, select your Target Group and confirm that all three tasks have been registered as "targets" with a "healthy" state

13.  Go to AWS -> EC2 -> Load Balancers and select your Load Balancer

14. Copy DNS of your Load Balancer

15. Execute test request (just adjust URL to DNS of your Load Balancer)

Create test measurement

```bash
curl -vk 'http://myapp-lb-564621670.eu-central-1.elb.amazonaws.com/device/v1/test' \
--header 'Content-Type: application/json' \
-u testUser:welt \
--data '{
    "type": "test",
    "value": -510.190
}'
```

Retrieve mesurements

```bash
curl -vk http://myapp-lb-564621670.eu-central-1.elb.amazonaws.com/device/v1/test -u testUser:welt
```

# Deploy application with GitHub
First, please set secrets (credentials) in AWS Secrets Manager:
```json
{
  "backend": {
    "security": {
      "users": [
        {
          "username": "userEMEATest",
          "password": "$2a$10$uKw9ORqCF.qA3p6woHCgmeGW0jFuU9AstYhl61Uw8RTQ5AaZCfuru",
          "roles": "USER"
        }
      ]
    }
  }
}
```

You also need to update **task.json** and replace **<<TODO: set ARN of secrets manager>>** with ARN of your Secrets Manager.

Then, please set **BACKEND_EMEA_TEST_SMOKETEST_BACKEND_PASSWORD** repository secret to "welt", as this is the password for the above test user, that will be used for smoke tests.

Make sure that you have also:
* Replaced <<ACCOUNT_ID>> in the whole project (replace all in all files) with your AWS Account ID.
   * **Make sure to push your changes to your forked repository!**
* Set AWS credentials in GitHub Settings

Go to Settings -> Secrets and variables and setup AWS credentials:
* **BACKEND_EMEA_TEST_AWS_KEY**
* **BACKEND_EMEA_TEST_AWS_SECRET**

Finally, please run **Multibranch pipeline** to deploy application to previously created infrastructure.
