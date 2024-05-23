# AWS Training - Terraform Hands-on
## Create bucket
Documentation:
* https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket

1. Go to ```/dad/main.tf```

Implement:


```hcl
# Sets up an S3 bucket to store the remote states of all modules
resource "aws_s3_bucket" "remote_state" {
  bucket = var.name
}

resource "aws_s3_bucket_versioning" "remote_state_versioning" {
  bucket = aws_s3_bucket.remote_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_lifecycle_configuration" "remote_state_lifecycle_configuration" {
  bucket = aws_s3_bucket.remote_state.id
  rule {
    id = "delete"
    status = "Enabled"

    noncurrent_version_expiration {
      noncurrent_days = 365
    }
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "remote_state_server_side_encryption_configuration" {
  bucket = aws_s3_bucket.remote_state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "public_access" {
  bucket = aws_s3_bucket.remote_state.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

2. Test
