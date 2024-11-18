![](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya/terraform-aws-s3)&nbsp;![](https://img.shields.io/github/last-commit/subhamay-bhattacharyya/terraform-aws-s3)&nbsp;![](https://img.shields.io/github/release-date/subhamay-bhattacharyya/terraform-aws-s3)&nbsp;![](https://img.shields.io/github/repo-size/subhamay-bhattacharyya/terraform-aws-s3)&nbsp;![](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya/terraform-aws-s3)&nbsp;[](https://img.shields.io/github/issues/subhamay-bhattacharyya/terraform-aws-s3)&nbsp;![](https://img.shields.io/github/languages/top/subhamay-bhattacharyya/terraform-aws-s3)&nbsp;![](https://img.shields.io/github/commit-activity/m/subhamay-bhattacharyya/terraform-aws-s3)&nbsp;![](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/4689677f7b4d68f0777a3b3959bbd04f/raw/terraform-aws-s3.json?)

# Terraform AWS S3 Module

This Terraform module creates an S3 bucket on AWS with various configurations.

## Usage

```hcl
module "s3_bucket" {
  source  = "app.terraform.io/subhamay-bhattacharyya/s3-bucket/aws"
  version = "1.0.0"

  aws-region               = "us-east-1"
  project-name             = "your-project-name"
  environment-name         = "devl"
  bucket-base-name         = "your-bucket-base-name"
  versioning-enabled       = true
  sse-algorithm            = "AES256"
  kms-master-key-id        = null
  s3-lifecycle-rules       = null
  bucket-policy-json       = null
  s3-tags                  = null
  ci-build                 = "your-ci-build-string"
}
```

#### Note

- To use default encryption pass `null` or `AES256` for sse-algorithm and `null` for `kms-master-key-id`
- To use SSE-KMS encryption pass `aws:kms` for sse-algorithm and kms key arn `kms-master-key-id`
- To create a bucket without any lifecycle rule pass `null` for `s3-lifecycle-rules`
- To create or update a bucket without versioning enabled pass `false` for `versioning-enabled`
- To add custom bucket tags pass a map as Key, Value pairs.
- To create or update a bucket with bucket policy pass `bucket-policy-json` with `data.aws_iam_policy_document.s3_bucket_policy.json` and define the policy in the data block as

```hcl
data "aws_caller_identity" "current" {}
data "aws_iam_policy_document" "s3_bucket_policy" {
  statement {
    principals {
      type        = "AWS"
      identifiers = [data.aws_caller_identity.current.account_id]
    }

    actions = [
      "s3:GetObject",
      "s3:ListBucket",
    ]

    resources = [
      "arn:aws:s3:::${var.project-name}-${var.bucket-base-name}-${var.environment-name}-${var.aws-region}${var.ci-build}",
      "arn:aws:s3:::${var.project-name}-${var.bucket-base-name}-${var.environment-name}-${var.aws-region}${var.ci-build}/*",
    ]
  }
}
```

## Inputs

| Name               | Description                                                     | Type        | Default  | Required |
| ------------------ | --------------------------------------------------------------- | ----------- | -------- | -------- |
| bucket-base-name   | The name of the S3 bucket                                       | string      | n/a      | yes      |
| versioning-enabled | Whether versioning is enabled for the S3 bucket                 | bool        | true     | no       |
| encryption-enabled | Whether server-side encryption is enabled for the S3 bucket     | bool        | true     | no       |
| kms-master-key-id  | The AWS KMS master key ID used for the SSE-KMS encryption       | string      | n/a      | no       |
| sse-algorithm      | The server-side encryption algorithm to use (AES256 or aws:kms) | string      | "AES256" | no       |
| s3-lifecycle-rules | A list of lifecycle rules for the S3 bucket                     | map(object) | {}       | no       |
| bucket-policy-json | The JSON policy to apply to the S3 bucket                       | string      | n/a      | no       |
| s3-tags            | S3 Bucket tags                                                  | map(string) | {}       | no       |
| ci-build           | CI build identifier                                             | string      | n/a      | no       |

## Outputs

| Name                        | Description                               |
| --------------------------- | ----------------------------------------- |
| bucket-arn                  | The ARN of the S3 bucket                  |
| bucket-name                 | The name of the S3 bucket                 |
| bucket-region               | The region of the S3 bucket               |
| bucket-domain-name          | The domain name of the S3 bucket          |
| bucket-regional-domain-name | The regional domain name of the S3 bucket |
| tags-all                    | All tags assigned to the S3 bucket        |
