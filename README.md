# Shifting Left: Implementing Static Analysis on Terraform with Checkov

Checkov <https://checkov.io> is a relatively new open source tool from the Bridgecrew <https://bridgecrew.io/>, it can check you infrastructure code against are large number of builtin checks and also expands with your own tests.

How do you actually start to implement this in your development pipeline?

I'm a big fan of using the pre-commit framework <https://pre-commit.com/>, this helps to find issues before you commit and share with others, It makes you look good and I'll take that any day.

So to start, create a new repository at your CLI:

```shell
mkdir shift-left
cd shift-left
git init
```

That's now a local repository, I have a standard pre-commit config that I use but this is a cut down for demonstration, add it to the root of your repo as **.pre-commit-config.yaml**:

```yaml
---
# yamllint disable rule:line-length
default_language_version:
  python: python3
repos:
  - repo: git://github.com/jameswoolfenden/pre-commit
    rev: v0.1.33
    hooks:
      - id: terraform-fmt
      - id: checkov-scan
        language_version: python3.7
```

Then install the hooks:

```shell
git add -A
pre-commit install
```

The first hook is a basic run of the builtin Terraform fmt command.
The second, hooks up to scan Terraform changes with Checkov, Checkov does not need to be installed and it works on Windows, linux and macs.

The next step is add some actual Terraform code, it's basic, add **aws_s3_bucket.holey.tf**:

```terraform
resource "aws_s3_bucket" "holey" {
  bucket = "my-tf-holey-bucket"
}
```

Add that:

```shell
git add aws_s3_bucket.holey.tf
```

And try to commit the change:

```shell
$ git commit -am "With Checkov"
terraform-fmt............................................................Passed
checkov..................................................................Failed
- hook id: checkov-scan
- exit code: 1

       _               _
   ___| |__   ___  ___| | _______   __
  / __| '_ \ / _ \/ __| |/ / _ \ \ / /
 | (__| | | |  __/ (__|   < (_) \ V /
  \___|_| |_|\___|\___|_|\_\___/ \_/

by bridgecrew.io | version: 1.0.426

terraform scan results:

Passed checks: 2, Failed checks: 4, Skipped checks: 0

Check: CKV_AWS_20: "S3 Bucket has an ACL defined which allows public READ access."
        PASSED for resource: aws_s3_bucket.holey
        Guide: https://docs.bridgecrew.io/docs/s3_1-acl-read-permissions-everyone
        File: /aws_s3_bucket.holey.tf:1-3


Check: CKV_AWS_57: "S3 Bucket has an ACL defined which allows public WRITE access."
        PASSED for resource: aws_s3_bucket.holey
        Guide: https://docs.bridgecrew.io/docs/s3_2-acl-write-permissions-everyone
        File: /aws_s3_bucket.holey.tf:1-3


Check: CKV_AWS_18: "Ensure the S3 bucket has access logging enabled"
        FAILED for resource: aws_s3_bucket.holey
        Guide: https://docs.bridgecrew.io/docs/s3_13-enable-logging
        File: /aws_s3_bucket.holey.tf:1-3

                1 | resource "aws_s3_bucket" "holey" {
                2 |   bucket = "my-tf-holey-bucket"
                3 | }

Check: CKV_AWS_19: "Ensure all data stored in the S3 bucket is securely encrypted at rest"
        FAILED for resource: aws_s3_bucket.holey
        Guide: https://docs.bridgecrew.io/docs/s3_14-data-encrypted-at-rest
        File: /aws_s3_bucket.holey.tf:1-3

                1 | resource "aws_s3_bucket" "holey" {
                2 |   bucket = "my-tf-holey-bucket"
                3 | }

Check: CKV_AWS_52: "Ensure S3 bucket has MFA delete enabled"
        FAILED for resource: aws_s3_bucket.holey
        File: /aws_s3_bucket.holey.tf:1-3

                1 | resource "aws_s3_bucket" "holey" {
                2 |   bucket = "my-tf-holey-bucket"
                3 | }

Check: CKV_AWS_21: "Ensure all data stored in the S3 bucket have versioning enabled"
        FAILED for resource: aws_s3_bucket.holey
        Guide: https://docs.bridgecrew.io/docs/s3_16-enable-versioning
        File: /aws_s3_bucket.holey.tf:1-3

                1 | resource "aws_s3_bucket" "holey" {
                2 |   bucket = "my-tf-holey-bucket"
                3 | }

```

So as you can see Checkov has highlighted a number of issues with your code. 6 tests and 4 failing, everything from being a Public bucket to no encryption, this may or may not fit with your use case.

The rest is up to you, fix those settings or suppress as appropriate.

For my S3 module I ended up with this, plus appropriate defaults for my variables:

```terraform
resource "aws_s3_bucket" "bucket" {
  acl    = var.s3_bucket_acl
  bucket = var.s3_bucket_name
  policy = var.s3_bucket_policy

  force_destroy = var.s3_bucket_force_destroy
  #checkov:skip=CKV_AWS_18: "Ensure the S3 bucket has access logging enabled"
  versioning {
    enabled    = var.versioning
    mfa_delete = var.mfa_delete
  }

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = var.sse_algorithm
      }
    }
  }

  tags = var.common_tags
}
```

and from my **variables.tf**

```terraform
...
variable "s3_bucket_acl" {
  default     = "private"
  description = "Acl on the bucket"
  type        = string
}
...
```

The code of this article is available here: <https://github.com/JamesWoolfenden/shift-left>, if you want to see a full finished version of an S3 bucket in Terraform that complies and suppresses with Checkov:
<https://github.com/JamesWoolfenden/terraform-aws-s3>
