---
title: "Summary and Resource Cleanup"
date: 2026-07-30
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

The workshop has presented:

- The data pipeline and S3 artifact layout.
- Five DynamoDB tables and their access patterns.
- ALS training, evaluation, and the promotion gate.
- A SageMaker Processing Job.
- The FastAPI provider, cache, and failure paths.
- Assumptions for deploying the application on EC2.
- Least-privilege IAM boundaries.

## Resources That May Incur Costs

- SageMaker real-time endpoint and endpoint instance.
- SageMaker Processing Jobs while they are running.
- EC2 instance and attached storage.
- DynamoDB requests, backups, and tables.
- S3 current objects, noncurrent versions, and multipart uploads.
- CloudWatch Logs.
- Data transfer.
- Optional network resources not described in the repository.

## Resource Cleanup Order

![Dependency order for cleaning up AWS resources](/images/5-Workshop/5.6-Cleanup/cleanup-dependency-flow.jpg)

*Clean up resources in dependency order to preserve required data and avoid leaving resources that still reference one another.*

Proceed in this order:

1. Stop traffic, CI deployment, and the retraining scheduler.
2. Export or retain the logs, reports, model manifests, and data that must be preserved.
3. Stop active SageMaker jobs.
4. Delete the Endpoint before the EndpointConfig and Model if the owner confirms that these resources belong to the workshop.
5. Stop or terminate EC2 according to the retention policy.
6. Export DynamoDB data that must be preserved before deleting test items or tables.
7. Delete S3 current objects, versions, and multipart uploads before deleting the bucket.
8. Detach managed/inline policies before deleting IAM roles.
9. Review Billing, Cost Explorer, and the resource inventory.

## Data That May Be Lost Permanently

- User accounts and password hashes.
- `UserInteractions` used for retraining.
- `RecommendationCache`.
- Movie catalog and popularity ranking.
- Raw/processed datasets.
- Model artifacts, manifests, and evaluation history.
- S3 object versions.
- Logs and audit evidence.

## Read-Only Inventory

The following commands only list resources; they do not perform cleanup:

```bash
aws sagemaker list-endpoints \
  --region "<AWS_REGION>"

aws sagemaker list-processing-jobs \
  --region "<AWS_REGION>"

aws ec2 describe-instances \
  --region "<AWS_REGION>" \
  --filters Name=instance-state-name,Values=running

aws dynamodb list-tables \
  --region "<AWS_REGION>"

aws s3api list-objects-v2 \
  --bucket "<S3_BUCKET_NAME>" \
  --max-items 10
```
