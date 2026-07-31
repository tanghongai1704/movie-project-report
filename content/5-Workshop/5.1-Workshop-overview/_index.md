---
title: "Overall Architecture and Processing Flows"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

The system consists of a **React/Vite** frontend, a **FastAPI** backend, five DynamoDB tables, an S3 bucket containing data and artifacts, a recommendation provider, and two retraining options: local/EC2 execution or a SageMaker Processing Job.

## Overall Architecture

![Overall architecture](/images/5-Workshop/5.1-Workshop-overview/overall_architecture.png)

*Overall architecture of the movie recommendation system on AWS.*

## Application Processing Flow

1. The browser calls centralized frontend services.
2. `apiClient` attaches the base URL, JSON headers, and a JWT for endpoints that require authentication.
3. The FastAPI router authenticates the request and calls the corresponding service.
4. The service applies business rules and calls a repository or `RecommendationProvider`.
5. The repository performs DynamoDB operations; the provider invokes the SageMaker Endpoint.
6. The returned `movie_id` values are enriched with metadata from the `Movies` table before the response is returned to the frontend.

### Guest Flow

The guest flow reads only `PopularMovies`, then uses `BatchGetItem` to retrieve metadata from `Movies`. This flow does not invoke SageMaker.

### Personalized Recommendation Flow

The backend checks `RecommendationCache` first. If the cache entry is still valid, the result is returned without invoking the endpoint. On a cache miss, the backend builds the request context, invokes the SageMaker Endpoint, validates the response, stores the cache entry on a best-effort basis, and enriches the movie metadata.

![Recommendation request flow through the cache and SageMaker Endpoint](/images/5-Workshop/5.1-Workshop-overview/backend-request-flow.jpg)

*Recommendation request flow: check the cache, invoke SageMaker on a cache miss, and retrieve metadata from the Movies table.*

## Training Flow

1. Kaggle CSV files are profiled, cleaned, and mapped from MovieLens IDs to TMDB movie IDs.
2. The pipeline creates content features, interactions, and time-based training, validation, and test datasets.
3. The ALS model is trained and evaluated offline.
4. The promotion gate determines whether the `LATEST.json` pointer should be updated.
5. Artifacts and reports are synchronized to S3.
6. Interactions from the operating environment can be exported from DynamoDB for the next retraining run.

## Role of Each Service

| Component | Role |
|---|---|
| Amazon S3 | Durable storage for datasets, model artifacts, and reports |
| Amazon DynamoDB | Stores metadata, accounts, interactions, and request-time cache entries |
| SageMaker Processing Job | Runs batch retraining |
| SageMaker Endpoint | Invocation target for the recommendation provider in the backend |
| Amazon EC2 | Runs the web application with Docker (React + FastAPI) and the retraining schedule |
| AWS IAM | Separates deployment, runtime, and SageMaker execution permissions according to least privilege |
| Amazon VPC & Internet Gateway | Provides the network boundary (public subnet) and Internet routing for public HTTP access and deployment over SSH from GitHub Actions |
| Amazon CloudWatch | Collects application container logs and operational metrics for SageMaker Processing Jobs |
| AWS Budgets | Tracks cloud costs against the budget limit ($200 AWS credit guardrail) |
