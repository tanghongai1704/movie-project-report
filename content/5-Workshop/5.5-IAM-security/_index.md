---
title: "IAM, Least Privilege Principle, and Security"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

The repository uses the default boto3 credential provider chain:

- Developers should use AWS IAM Identity Center or configured profiles.
- EC2 hosts use IAM instance profiles.
- SageMaker execution roles are assigned to processing jobs.

Exact role names, JSON policy definitions, trust relationships, and resource ARNs are not stored within this repository.

![Credential flow between developers, EC2, and AWS services](/images/5-Workshop/5.5-IAM-security/security-credential-flow.png)

*Developer profiles or EC2 instance profiles provide credentials to boto3, while the JWT secret is managed separately for FastAPI authentication.*

## Principal Permission Matrix

| Principal | Required Permissions | Resource Scope |
|---|---|---|
| Backend EC2 role | `sts:GetCallerIdentity` | Current identity |
| Backend EC2 role | DynamoDB describe/get/batch-get/put/query/scan per code path | ARNs of five tables and relevant indexes |
| Backend EC2 role | S3 list for startup validation | Single bucket, restricted to prefixes where applicable |
| Backend EC2 role | `sagemaker:DescribeEndpoint`, `sagemaker:InvokeEndpoint` | Single endpoint resource |
| S3/ML operator | S3 list/get/put | Data, model, and report prefixes |
| Processing submitter | Create/describe Processing Jobs, `iam:PassRole` | Job namespace and execution role ARN |
| SageMaker execution role | S3 list/get/put, CloudWatch Logs | Specific prefixes and log groups |
| GitHub deployer | SSH credential in GitHub Secrets | Single EC2 host |

{{% notice warning %}}
Do not use `Action: "*"` or `Resource: "*"` wildcards simply to force components to execute. Production IAM policies must be thoroughly reviewed by security owners.
{{% /notice %}}

## Least Privilege Observations

- Backend HTTP runtime roles do not require `DeleteItem` permissions, as no delete API endpoints currently exist.
- S3 upload and download permissions should belong to dedicated tooling roles rather than broadening web runtime permissions.
- The `Users` table currently requires table scan operations for case-insensitive login checks due to the absence of an identity GSI.
- The principal submitting SageMaker jobs and the SageMaker execution role are distinct identity entities.
- `iam:PassRole` permissions must be restricted strictly to the designated execution role ARN.

## Application Security Controls

- Passwords are hashed using PBKDF2-HMAC-SHA256 with random salt strings and configurable iteration counts.
- JWT HS256 validation verifies signatures, issuer claims, audience claims, and token expiration.
- Protected actions attempted by guests return `401 Unauthorized`.
- Users who have not completed onboarding receive `403 Forbidden` when attempting recommendation requests.
- The frontend stores access tokens in `localStorage`; cross-site scripting (XSS) protections must be enforced.
- User logout endpoints do not server-side revoke JWT tokens; endpoints return `204 No Content`.

## Positive Permission Test

With proper IAM role attachments verified on backend services:

```bash
aws sts get-caller-identity \
  --region "<AWS_REGION>"

aws dynamodb describe-table \
  --table-name "<AUTHORIZED_TABLE_NAME>" \
  --region "<AWS_REGION>"

aws sagemaker describe-endpoint \
  --endpoint-name "<SAGEMAKER_ENDPOINT_NAME>" \
  --region "<AWS_REGION>"
```

Pass criteria: all three authorized operations complete successfully.



<!-- IMAGE-5.5-01: IAM role policy restricted to specific resource ARNs with sensitive identifiers redacted. -->

## Safe Negative Permission Test

Security administrators must provide an explicitly approved out-of-scope test resource for negative testing:

```bash
aws dynamodb describe-table \
  --table-name "<APPROVED_OUT_OF_SCOPE_TEST_TABLE>" \
  --region "<AWS_REGION>"
```

Expected result: `AccessDeniedException`.

`ResourceNotFoundException` does not satisfy least privilege validation, as the target resource may simply not exist.

<!-- IMAGE-5.5-02: AccessDeniedException console response when accessing an out-of-scope test resource. -->


