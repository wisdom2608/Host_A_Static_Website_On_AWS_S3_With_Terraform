**Host A Static Website On AWS S3**

How to host a static website on aws S3 bucket using terraform 

Here’s a complete Terraform script to host a static website on an AWS S3 bucket. This will:

1.	*Create an S3 bucket.*
2.	*Enable static website hosting*.
3.	*Upload static files (optional)*.
4.	*Make the bucket publicly accessible*.
5.	*Output the website URL*.

 ✅ **Prerequisites**
 
 •	AWS CLI configured (aws configure)
 
 •	Terraform installed
 
 •	A directory containing:
 
 •	Your static files (e.g., index.html)
 
 •	Terraform files (main.tf, variables.tf, outputs.tf)

📁 **Folder Structure Example**

 ```hcl
static-site/
│
├── main.tf
├── variables.tf
├── outputs.tf
└── website-files/
    ├── index.html
    └── error.html
 ```
🛠️ **1. main.tf**

```hcl

provider "aws" {
  region = "us-east-1"  # You can change this
}

resource "aws_s3_bucket" "static_site" {
  bucket = var.bucket_name

  website {
    index_document = "index.html"
    error_document = "error.html"
  }

  versioning {
    enabled = true  # 👈 Enables versioning
  }

  tags = {
    Name        = "StaticSiteBucket"
    Environment = "Dev"
  }
}

resource "aws_s3_bucket_public_access_block" "public_block" {
  bucket = aws_s3_bucket.static_site.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_policy" "allow_public_read" {
  bucket = aws_s3_bucket.static_site.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = "*"
        Action = ["s3:GetObject"]
        Resource = "${aws_s3_bucket.static_site.arn}/*"
      }
    ]
  })
}

resource "aws_s3_object" "index" {
  bucket = aws_s3_bucket.static_site.id
  key    = "index.html"
  source = "${path.module}/website-files/index.html"
  content_type = "text/html"
  acl    = "public-read"
}

resource "aws_s3_object" "error" {
  bucket = aws_s3_bucket.static_site.id
  key    = "error.html"
  source = "${path.module}/website-files/error.html"
  content_type = "text/html"
  acl    = "public-read"
}

```

📄 **2. variables.tf**

```hcl
variable "bucket_name" {
  description = "The name of the S3 bucket"
  type        = string
}
```
📤 **3. outputs.tf**

```hcl
output "website_url" {
  description = "The URL of the static website"
  value       = aws_s3_bucket.static_site.website_endpoint
}
```
🌐 **4. Your index.html**

Make sure there’s an index.html in the same directory:

```html
<!DOCTYPE html>
<html>
  <head><title>My Static Website</title></head>
  <body>
    <h1>Hello from S3 Static Website!</h1>
  </body>
</html>
```

**Optional: Add an error.html.**

🚀 **Deployment Steps**

```bash
terraform init
terraform plan -var="bucket_name=my-unique-static-site-bucket"
terraform apply -var="bucket_name=my-unique-static-site-bucket"
```
📝 **Bucket names must be globally unique, or you’ll get an error.**

Visit the website using the output:
```bash
Outputs:

website_url = http://my-unique-static-site-bucket.s3-website-us-east 1.amazonaws.com
```

🛡️ **Optional Enhancements**

•	Use Route 53 to map a custom domain.
•	Add CloudFront for HTTPS and CDN.
•	Use Terraform modules for better reusability.


