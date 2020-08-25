# Shifting Left: Implementing Static Analysis on Terraform code with Checkov

Checkov is a relatively new open source tool from the Bridgecrew, it can check you infrastructure code against are large number of built but expandable in security tests.

How do you implement this in your development pipeline?

I'm a big fan of using the pre-commit framework, this helps to find issues before you commit and share with others, It makes you look good and I'll take that gladly.

Create a new repository at your CLI:

```
mkdir shift-left
cd shift-left
git init
```

That's now a local repository, I have a standard pre-commit config that I use but this is a cut down for demostration, add it to the root of your repo as **.pre-commit-config.yaml**:

```
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

```
git add -A
pre-commit install
```

The next step is add some actual Terraform code, it's basic, add **aws_s3_bucket.holey.tf**:

```terraform
resource "aws_s3_bucket" "holey" {
  bucket = "my-tf-holey-bucket"
}
```

Add that:
```
git add aws_s3_bucket.holey.tf
```

And try to commit the change:
```
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

So as you can see checkov has highlighted a number of issues with your code. The rest is up to you, fix or suppress as appropriate.





