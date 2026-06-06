# Infrastructure as Code — Serverless AWS Stack (Terraform)

Terraform configuration that provisions a complete **serverless web stack** on AWS: a
static website behind a CDN with HTTPS, plus a serverless backend for handling contact-form
submissions.

## What it provisions

- **Static hosting** — S3 bucket configured for website hosting.
- **CDN + HTTPS** — CloudFront distribution with an ACM TLS certificate
  (DNS-validated via Route 53) and HTTP→HTTPS redirection.
- **DNS** — Route 53 hosted zone and records, including an alias to the CloudFront
  distribution.
- **Serverless backend** — a Lambda function fronted by an HTTP API Gateway
  (`POST /submit`) that stores submissions in DynamoDB.
- **Data store** — DynamoDB table (pay-per-request) keyed by email.
- **Least-privilege IAM** — a dedicated execution role and scoped policy for the Lambda
  (DynamoDB writes, SES email, CloudWatch logs).

## Tech stack

- Terraform
- AWS: S3, CloudFront, ACM, Route 53, DynamoDB, Lambda, API Gateway, IAM

## Architecture

```
            ┌──────────────┐
 Browser ──▶│  CloudFront  │──▶ S3 (static site)
            └──────────────┘
 Browser ──▶ API Gateway ──▶ Lambda ──▶ DynamoDB
                                   └──▶ SES (email)
```

## Getting started

Provide values for the declared variables (region, bucket name, domain name, table name,
Lambda settings) via a `terraform.tfvars` file or CLI flags, then:

```bash
terraform init
terraform plan
terraform apply
```

> Note: Requires a Lambda deployment package (`function.zip`) and a domain whose
> nameservers point at the created Route 53 zone for ACM validation to complete.
