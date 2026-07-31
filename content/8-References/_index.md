---
title: "References"
date: 2026-07-31
weight: 8
chapter: false
pre: " <b> 8. </b> "
---

This section collects the source code, demo, official AWS documentation, and repository files used as evidence for the movie recommendation workshop.

## Project Resources

| Resource | Link |
|---|---|
| Movie recommendation source code | [GitHub - movie-recommendation-system](https://github.com/CaPPok/movie-recommendation-system) |
| Report and workshop repository | [GitHub - movie-project-report](https://github.com/tanghongai1704/movie-project-report) |
| Published report website | [GitHub Pages - movie-project-report](https://tanghongai1704.github.io/movie-project-report/) |

## Demo

- [Google Drive - Movie recommendation system demo](https://drive.google.com/drive/folders/1TNqHmVXZxYamXQ_ZqLBBzCpeKkqFaSAn?usp=sharing)

{{% notice note %}}
Make sure viewers can access the Google Drive folder before publishing the report. Never include access tokens, secret keys, passwords, or temporary signed URLs in this documentation.
{{% /notice %}}

## Official AWS Documentation

### Amazon S3 and DynamoDB

- [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [Creating an Amazon S3 bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html)
- [Amazon DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [Create a DynamoDB table](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/getting-started-step-1.html)
- [Configure TTL in DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/time-to-live-ttl-how-to.html)
- [Point-in-time recovery for DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery_Howitworks.html)

### Amazon SageMaker AI

- [Amazon SageMaker AI Developer Guide](https://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html)
- [SageMaker execution roles](https://docs.aws.amazon.com/sagemaker/latest/dg/sagemaker-roles.html)
- [CreateModel API](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateModel.html)
- [CreateEndpointConfig API](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateEndpointConfig.html)
- [Deploy a model for real-time inference](https://docs.aws.amazon.com/sagemaker/latest/dg/realtime-endpoints-deploy-models.html)

### IAM and Amazon EC2

- [Security best practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)

The AWS documentation links above were last verified on **July 31, 2026**.

## Workshop Source Evidence

| Source repository path | Evidence used in the workshop |
|---|---|
| `backend/app/aws/infrastructure.py` | Defines how the backend accesses AWS resources. |
| `docs/aws/dynamodb.md` | Documents the DynamoDB schema and access patterns. |
| `docs/aws/aws-setup.md` | Provides AWS environment configuration guidance. |
| `configs/data_pipeline.yaml` | Configures the data processing pipeline. |
| `configs/model_serving.yaml` | Configures the model and serving behavior. |
| `configs/aws.yaml` | Defines the Region, S3 prefixes, SageMaker settings, and promotion criteria. |
| `scripts/sagemaker_retrain_job.py` | Launches or dry-runs the SageMaker retraining job. |
| `scripts/test_sagemaker_endpoint.py` | Describes and tests the SageMaker Endpoint. |

## Maintenance Notes

- Prefer permanent links, releases, or fixed tags so references do not change unexpectedly.
- Record the commit SHA or release version used as workshop evidence.
- Verify access to the source repository and demo folder before publication.
- Recheck all links before each report update.
