---
title: "Workshop"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

This workshop presents the architecture and operational workflow of a movie recommendation system built with **React**, **FastAPI**, and AWS services. The system uses:

- **Amazon DynamoDB** to store movie information, accounts, user behavior, and recommendation cache entries.
- **Amazon S3** to store raw data, processed data, training datasets, model artifacts, and evaluation reports.
- **Amazon SageMaker Processing Job** to run the on-demand retraining workflow.
- **Amazon SageMaker Endpoint** as the real-time inference target called by the backend.
- **Amazon EC2** to run the application with Docker Compose.
- **AWS IAM** to separate the permissions of deployers, the application, and SageMaker.

![Main interface of the Streamverse system](/images/5-Workshop/ui-home-page.png)

*Main interface of the movie browsing and content recommendation application.*

## Problem Being Solved

The system supports three primary scenarios:

1. **Unauthenticated guests** browse the popular movie list.
2. **New users** select their preferred genres during onboarding.
3. **Returning users** receive recommendations based on their interaction history, cache entries, and the recommendation provider.

## Confirmed Scope

- DynamoDB contains five logical tables: `Movies`, `PopularMovies`, `Users`, `UserInteractions`, and `RecommendationCache`.
- S3 is divided into zones for raw data, processed data, training datasets, inference data, models, evaluation reports, and interaction exports.
- The ML pipeline uses **implicit ALS** for collaborative filtering and includes evaluation and a promotion gate.
- The backend includes code that invokes a SageMaker real-time endpoint.
- GitHub Actions deploys the application to an existing EC2 host.
- The application code uses the default AWS SDK credential provider chain.

## Learning Outcomes

After completing this workshop, you will be able to:

- Explain the data flow, training flow, and request-time inference flow.
- Verify the schemas of the five DynamoDB tables and the S3 prefix structure.
- Run the data pipeline, validation, and a dry run of the retraining process.
- Train and evaluate the model locally or submit a SageMaker Processing Job when the required permissions are available.
- Start the application and test the guest, authentication, and interaction flows.
- Diagnose cache hits, endpoint failures, and permission errors.
- Plan resource cleanup in the correct dependency order.

## Workshop Contents

1. [Overall architecture and processing flows](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequisites/)
3. [Data layer with S3 and DynamoDB](5.3-Data-layer/)
4. [Recommendation pipeline](5.4-Recommendation-pipeline/)
5. [IAM and security](5.5-IAM-security/)
6. [Summary and resource cleanup](5.6-Cleanup/)
