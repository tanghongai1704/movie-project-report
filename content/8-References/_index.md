---
title: "References"
date: 2026-07-31
weight: 8
chapter: false
pre: " <b> 8. </b> "
---

This section collects the documentation, repositories, and supporting resources used by the movie recommendation report and workshop.

## Project Resources

| Resource | Status | Link or note |
|---|---|---|
| Report and workshop repository | Available | [GitHub - movie-project-report](https://github.com/tanghongai1704/movie-project-report) |
| Published report website | Available | [GitHub Pages - movie-project-report](https://tanghongai1704.github.io/movie-project-report/) |
| Movie recommendation source code | URL not provided | Replace `<RECOMMENDATION_SOURCE_REPOSITORY_URL>` with the actual repository. If the frontend, backend, and ML components use separate repositories, list each one individually. |
| Demo video | URL not provided | Replace `<DEMO_VIDEO_URL>` with an accessible YouTube, Google Drive, or approved hosting link. |

{{% notice note %}}
Verify repository and demo-video permissions before publication. Never include access tokens, secret keys, passwords, or temporary signed URLs in this report.
{{% /notice %}}

## Official AWS Documentation

- [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [Amazon DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [Amazon SageMaker AI Developer Guide](https://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html)
- [Security best practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)

The AWS documentation links above were last verified on **July 31, 2026**.

## Workshop Source Evidence

The following paths belong to the recommendation system source repository and should be connected to source-code URLs when that repository is published:

- `backend/app/aws/infrastructure.py`: Backend AWS resource access definitions.
- `docs/aws/dynamodb.md`: DynamoDB schema and access patterns.
- `docs/aws/aws-setup.md`: AWS configuration guidance.
- `configs/data_pipeline.yaml`: Data processing configuration.
- `configs/model_serving.yaml`: Model and serving configuration.
- `configs/aws.yaml`: Region, S3 prefixes, SageMaker settings, and promotion criteria.
- `scripts/sagemaker_retrain_job.py`: Retraining job launch and dry-run behavior.
- `scripts/test_sagemaker_endpoint.py`: SageMaker Endpoint description and invocation tests.

## Maintenance Notes

- Replace placeholder URLs as soon as the source repository and demo video are available.
- Prefer permanent links or fixed releases/tags so references do not silently change.
- Record the commit SHA or release version used as workshop evidence.
- Recheck all links before each report release.
