---
title: Workshop
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

This workshop presents the architecture and operational workflow of a movie recommendation system built with **React**, **FastAPI**, and AWS services. The system utilizes:

![Streamverse movie recommendation system home page](/images/5-Workshop/ui-home-page.png)

*The main interface of the movie streaming and recommendation application.*

- **Amazon DynamoDB** to store movie details, user accounts, user interaction history, and recommendation cache.
- **Amazon S3** to store raw data, processed data, training sets, model artifacts, and evaluation reports.
- **Amazon SageMaker Processing Job** to execute on-demand retraining pipelines.
- **Amazon SageMaker Runtime** as the real-time inference target for the backend.
- **Amazon EC2** to host and run the application via Docker Compose.
- **AWS IAM** to segregate permissions among deployer, application, and SageMaker execution roles.

## Problem Statement

The system addresses three main user scenarios:

1. **Unauthenticated Guests**: Browse popular movie lists.
2. **New Users**: Select preferred genres during the onboarding workflow.
3. **Returning Users**: Receive personalized recommendations based on interaction history, cache, and recommendation providers.

## Confirmed Scope

- DynamoDB has five logical tables: `Movies`, `PopularMovies`, `Users`, `UserInteractions`, and `RecommendationCache`.
- S3 is partitioned into logical zones for raw data, processed data, training sets, inference lookup, model artifacts, evaluation reports, and interaction exports.
- The ML pipeline uses **implicit ALS** for collaborative filtering, featuring offline evaluation and a promotion gate.
- The backend contains code to invoke a SageMaker real-time endpoint.
- GitHub Actions deploys the application to an existing EC2 server.
- The application code utilizes the default AWS SDK credential provider chain.

{{% notice warning %}}
The repository currently does not include Infrastructure as Code (IaC), DynamoDB data loading scripts, SageMaker serving handlers, model/EndpointConfig/Endpoint creation scripts, complete IAM configurations, EC2 provisioning workflows, or cleanup automation. Therefore, the workshop accurately describes existing components and highlights required manual steps.
{{% /notice %}}

## Learning Outcomes

Upon completing this workshop, you will be able to:

- Explain the data flow, training flow, and request-time inference flow.
- Inspect the schema of the five DynamoDB tables and S3 prefix structures.
- Execute data pipelines, validation, and dry-runs for model retraining.
- Train and evaluate models locally, or launch SageMaker Processing Jobs given proper IAM permissions.
- Launch the application and test guest, authentication, and interaction flows.
- Diagnose cache hits, endpoint errors, and permission failures.
- Formulate a resource cleanup plan adhering to strict dependency ordering.

## Workshop Table of Contents

1. [Overall Architecture and Workflow](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequisites/)
3. [Data Layer with S3 and DynamoDB](5.3-Data-layer/)
4. [Recommendation Pipeline](5.4-Recommendation-pipeline/)
5. [IAM and Security](5.5-IAM-security/)
6. [Summary and Resource Cleanup](5.6-Cleanup/)

{{% notice note %}}
Do not include real AWS credentials, JWT tokens, AWS Account IDs, actual ARNs, or real `.env` contents in the report or screenshots.
{{% /notice %}}
