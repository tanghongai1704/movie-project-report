---
title: "Verify and Prepare Storage Resources"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

The repository does not contain Terraform, CloudFormation, CDK, or an equivalent Infrastructure as Code tool. The following section explains how to create resources manually in the AWS Console for the workshop environment and then verify them with the Console and AWS CLI. For production, these steps should be converted to Infrastructure as Code and reviewed by the platform/security team.

## 1. Create Five DynamoDB Tables in the AWS Console

### 1.1. Select the Region and Naming Convention

1. Sign in to the AWS Console with the IAM user/role created for the workshop; do not use the root user for daily operations.
2. Select the correct `<AWS_REGION>` in the upper-right corner. The workshop's S3, DynamoDB, SageMaker, and EC2 resources should use the same Region; the current evidence uses `ap-southeast-1`.
3. Use an environment-specific prefix, such as `movie-rec-dev-`, and then update the backend `.env` file with the exact table names.

### 1.2. Create Each Table

1. Open **DynamoDB** → **Tables** → **Create table**.
2. Enter the table name, partition key, and sort key shown in the table below. All keys use the **String** type.
3. Under **Table settings**, select **Customize settings**.
4. Select **On-demand** for a workshop with unpredictable traffic. Use provisioned capacity only when traffic measurements and a clear capacity plan are available.
5. Keep DynamoDB's default encryption, or select a customer-managed KMS key if required by organizational policy.
6. Add at least these tags: `Project=movie-recommendation`, `Environment=<ENVIRONMENT>`, and `Owner=<OWNER>`.
7. Select **Create table**, wait until the status changes to `ACTIVE`, and repeat until all five tables have been created.

| Logical name | Suggested table name | Partition key | Sort key |
|---|---|---|---|
| Movies | `<ENV_PREFIX>-Movies` | `movie_id` (String) | None |
| PopularMovies | `<ENV_PREFIX>-PopularMovies` | `list_id` (String) | None |
| Users | `<ENV_PREFIX>-Users` | `user_id` (String) | None |
| UserInteractions | `<ENV_PREFIX>-UserInteractions` | `user_id` (String) | `interaction_key` (String) |
| RecommendationCache | `<ENV_PREFIX>-RecommendationCache` | `user_id` (String) | `scenario` (String) |

## 2. Verify the Five DynamoDB Tables

For each table name supplied through a secure channel, run:

```bash
aws dynamodb describe-table \
  --table-name "<MOVIES_TABLE_NAME>" \
  --region "<AWS_REGION>"
```

Repeat for the other four tables and confirm:

| Table | HASH key | RANGE key |
|---|---|---|
| `Movies` | `movie_id` | None |
| `PopularMovies` | `list_id` | None |
| `Users` | `user_id` | None |
| `UserInteractions` | `user_id` | `interaction_key` |
| `RecommendationCache` | `user_id` | `scenario` |

All tables must be in the `ACTIVE` state.

![Five DynamoDB tables in the Active state](/images/5-Workshop/5.3-Data-layer/5.3.1-provision-storage/dynamodb-tables.png)

*Five DynamoDB tables with their partition keys, sort keys, and Active status.*

## 3. Create an S3 Bucket in the AWS Console

1. Open **Amazon S3** → **General purpose buckets** → **Create bucket**.
2. Select **General purpose**, enter a globally unique `<S3_BUCKET_NAME>`, and select the correct `<AWS_REGION>`.
3. Under **Object Ownership**, keep **Bucket owner enforced** to disable ACLs.
4. Under **Block Public Access settings for this bucket**, keep **Block all public access** and all four subordinate options enabled.
5. Under **Bucket Versioning**, select **Enable**.
6. Add the `Project`, `Environment`, and `Owner` tags.
7. Under **Default encryption**, select **SSE-S3** for this workshop. If the organization requires control of its own keys, select SSE-KMS and add the corresponding KMS permissions.
8. Select **Create bucket**.
9. Open the newly created bucket → **Create folder** to create the logical prefixes: `raw/`, `processed/`, `training/`, `inference/`, `models/`, `evaluation/`, and `interaction-exports/`.

## 4. Verify the S3 Bucket

![S3 bucket for the movie recommendation system](/images/5-Workshop/5.3-Data-layer/5.3.1-provision-storage/s3-bucket.png)

*S3 bucket used to store datasets, model artifacts, and reports.*

![S3 bucket overview](/images/5-Workshop/5.3-Data-layer/5.3.1-provision-storage/s3-bucket-overview.png)

*The bucket is deployed in Region `ap-southeast-1`; the overview directly confirms the ARN and Region of the resource being verified.*

```bash
aws s3api head-bucket \
  --bucket "<S3_BUCKET_NAME>"

aws s3api get-public-access-block \
  --bucket "<S3_BUCKET_NAME>"

aws s3api get-bucket-encryption \
  --bucket "<S3_BUCKET_NAME>"

aws s3api get-bucket-versioning \
  --bucket "<S3_BUCKET_NAME>"
```

The bucket must exist and have appropriate Block Public Access and encryption settings. For the current bucket, Block Public Access is enabled, default encryption is SSE-S3, and versioning has the `Enabled` status. The application source code does not provision these settings automatically; lifecycle configuration must still be reviewed separately if it is used.

![S3 bucket Block Public Access settings](/images/5-Workshop/5.3-Data-layer/5.3.1-provision-storage/s3-block-public-access.png)

*`Block all public access` is `On`, preventing public access through ACLs and bucket policies.*

![S3 bucket default encryption settings](/images/5-Workshop/5.3-Data-layer/5.3.1-provision-storage/s3-bucket-encryption.png)

*Default server-side encryption uses Amazon S3 managed keys (SSE-S3).*

![S3 bucket versioning settings](/images/5-Workshop/5.3-Data-layer/5.3.1-provision-storage/s3-bucket-versioning.png)

*Bucket Versioning is `Enabled`; MFA delete is `Disabled`.*

## 6. Verify the Prefixes

Retrieve no more than one object to avoid reading unnecessary data:

```bash
aws s3api list-objects-v2 \
  --bucket "<S3_BUCKET_NAME>" \
  --prefix "<RAW_PREFIX>" \
  --max-items 1
```

Repeat with the `processed`, `training`, `inference`, `models`, `evaluation`, and `interaction exports` prefixes.

An empty prefix is not necessarily an error. `AccessDenied`, an incorrect Region, or a nonexistent bucket indicates an issue that must be resolved.

## 7. When Resources Do Not Exist

If a table or bucket is missing:

1. Stop the deployment step.
2. Record the required Region, key schema, billing mode, TTL, encryption, lifecycle, and IAM owner.
3. Ask the platform/security team to provide the resource or reviewed IaC.
