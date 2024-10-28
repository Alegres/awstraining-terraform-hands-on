# Content
- [Content](#content)
- [Preparation](#preparation)
- [01 Create S3 bucket](#01-create-s3-bucket)
- [02 Create remote state and lock](#02-create-remote-state-and-lock)
- [03 Create environments](#03-create-environments)
- [04 Create ECS (whole infrastructure)](#04-create-ecs-whole-infrastructure)
- [Application deployment](#application-deployment)
  - [Set basic auth credentials in Secrets Manager](#set-basic-auth-credentials-in-secrets-manager)
  - [Deploy application with manual commands](#deploy-application-with-manual-commands)
  - [Deploy application with GitHub](#deploy-application-with-github)


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

**ATTENTION!**
If you cannot find the ```.aws\credentials```, then please create it and make sure to set default & backend-test profiles.
Default profile should contain some dummy access and secret keys to avoid accident changes in the real environment (default) if no profile is provided.

**DO NOT USE ROOT USER CREDENTIALS! ​**
Instead, create admin user in IAM, assign him AdministratorAccess policy and generate credentials for this non-root user.​
**Tutorial:** https://capgemini-my.sharepoint.com/personal/maciej_bus_capgemini_com/_layouts/15/stream.aspx?id=%2Fpersonal%2Fmaciej%5Fbus%5Fcapgemini%5Fcom%2FDocuments%2FRecordings%2FTworzenie%20u%C5%BCytkownika%20technicznego%20AWS%20IAM%2Emkv&nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJTdHJlYW1XZWJBcHAiLCJyZWZlcnJhbFZpZXciOiJTaGFyZURpYWxvZy1MaW5rIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXcifX0&ct=1730129634719&or=OWA%2DNT%2DMail&cid=604c6fac%2Dedd5%2D2310%2D5d8d%2D9e2e78a6a600&ga=1&referrer=StreamWebApp%2EWeb&referrerScenario=AddressBarCopied%2Eview%2E0ff5e82d%2D7349%2D4572%2Da9cb%2Dfe31d3606a27

**Attention!**
If you do not have **choco**, please run PowerShell as an admin and request for the permission to run it as admin (send request to IT).
Then, use the following command to install **choco**:

```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('
https://community.chocolatey.org/install.ps1'))
```

# 01 Create S3 bucket
Documentation:
* https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket
* https://registry.terraform.io/providers/hashicorp/aws/latest/docs#provider-configuration
* https://developer.hashicorp.com/terraform/language/modules/sources#local-paths

First, please cheack out to the first branch:
```git checkout 01-create-s3-bucket```

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
  -var='region=eu-central-1'
```

8. Check if S3 bucket has been createt in AWS
9. Destroy of AWS infrastructure

```hcl
terraform destroy \
  -var='environment=emea' \
  -var='profile=backend-test' \
  -var='region=eu-central-1'
```

# 02 Create remote state and lock
**Attention!** Make sure that you set your unique bucket string in wrapper.properties file.

1. Checkout to branch

```
git checkout 02-create-remote-state-and-lock
```

2. Go to the given directory

```
cd aws-infrastructure/terraform​
```

3. Start the w2.sh script to provision remote state bucket

```
./w2.sh backend-test eu-central-1 common/general/create-remote-state-bucket apply
```

4. Check if above bucket has been created in AWS

5. Start the w2.sh script to provision DynamoDB locks table

```
./w2.sh backend-test eu-central-1 common/general/create-dynamo-lock apply
```

6. Check if above table has been created in AWS

7. Go to S3 and open your custom TFSTATE bucket.

8. Open .tfstate file for your DynamoDB table that was created and analyze its content.

# 03 Create environments
**Attention!**
Make sure that you keep your wrapper.properties file with your unique bucket string.
You must also make sure that ```<<Set your account ID>>``` placeholder in **outputs.tf** is adjusted and set to your account id.

1. Checkout to branch

```
git checkout 03-create-environments
```

2. Go to the given directory

```
cd aws-infrastructure/terraform
```

**Attention!**
Please first adjust ```<<Set your account ID>>``` placeholder in **outputs.tf** and set it to your account id.
3. Start w2.sh script to create environmental variables

```
./w2.sh backend-test eu-central-1 environments/backend-test/emea/eu-central-1/globals apply
```

4. Check if above environmental variables have been created/updated in S3 state bucket in AWS

5. Start w2.sh script to create VPC

```
./w2.sh backend-test eu-central-1 common/networking/vpc apply
```

6. Check if above VPC has been created in AWS

7. Start w2.sh script to create Security Groups

```
./w2.sh backend-test eu-central-1 common/networking/securitygroups apply
```

8. Check if above Security Groups have been created in AWS.

# 04 Create ECS (whole infrastructure)
**Attention!**
Make sure that you keep your wrapper.properties file with your unique bucket string.
You must also make sure that ```<<Set your account ID>>``` placeholder in **outputs.tf** is adjusted and set to your account id.

1. Checkout to branch

```
git checkout 04-create-ecs 
```

2. Go to given directory:​

```
cd aws-infrastructure/terraform​
```

3. Start the setup_new_region.sh script​

```
./setup_new_region.sh w2.sh backend-test eu-central-1 apply​ -auto-approve
```

Analyze carefully the output.​
Apply only changes that you understand, one-by-one!​

4. After all is done – check your AWS account and make sure that the ECS Fargate cluster was created

# Application deployment (Optional)
## Set basic auth credentials in Secrets Manager
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

You also need to update **task.json** and replace **<<TODO: set ARN of secrets manager>>** with ARN of your Secrets Manager. Push your changes.

## Deploy application with manual commands
If you do not have Docker, Maven and JAVA installed locally, then please setup EC2 environment in AWS.

How to create EC2 instance which will allow us to deploy image? (skip SNS permissions):
* https://github.com/Alegres/awstraining-basics-hands-on?tab=readme-ov-file#create-ec2-sns-topic-and-run-notification-script

and then follow the instructions on how to build & deploy app using this instance:
* https://github.com/Alegres/awstraining-basics-hands-on?tab=readme-ov-file#deploy-our-application

Otherwise, when deploying from your local machine:
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

## Deploy application with GitHub
You can also fork **awstraining-terraform** base repository to your account.

Then, please set **BACKEND_EMEA_TEST_SMOKETEST_BACKEND_PASSWORD** repository secret to "welt", as this is the password for the above test user, that will be used for smoke tests.

Make sure that you have also:
* Replaced <<ACCOUNT_ID>> in the whole project (replace all in all files) with your AWS Account ID.
   * **Make sure to push your changes to your forked repository!**
* Set AWS credentials in GitHub Settings

Go to Settings -> Secrets and variables and setup AWS credentials:
* **BACKEND_EMEA_TEST_AWS_KEY**
* **BACKEND_EMEA_TEST_AWS_SECRET**

Finally, please run **Multibranch pipeline** to deploy application to previously created infrastructure.

# Destroying infrastructure
1. Start the setup_new_region.sh script​ with destroy command

```
./setup_new_region.sh w2.sh backend-test eu-central-1 destroy​ -auto-approve
```
