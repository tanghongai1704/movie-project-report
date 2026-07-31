---
title: "Overall Architecture and Workflow"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

The system consists of a **React/Vite** frontend, a **FastAPI** backend, five DynamoDB tables, an S3 bucket for storing datasets and artifacts, recommendation providers, and two options for executing model retraining: locally/on EC2 or via SageMaker Processing Jobs.

## Overall Architecture

![Overall Architecture](/images/5-Workshop/5.1-Workshop-overview/overall_architecture.png)

*Overall architecture diagram of the Movie Recommendation System on AWS.*

## Application Request Workflows

1. The browser invokes centralized services in the frontend.
2. `apiClient` attaches base URL, JSON headers, and JWT tokens for authenticated endpoints.
3. The FastAPI router authenticates the request and invokes the corresponding service.
4. The service applies business rules and calls repositories or the `RecommendationProvider`.
5. Repositories perform DynamoDB operations; providers invoke SageMaker Endpoint.
6. `movie_id` references are enriched with metadata from the `Movies` table before being returned to the frontend.

### Guest Workflow

The guest workflow only reads from `PopularMovies`, then uses `BatchGetItem` to fetch movie metadata from `Movies`. This path does not invoke SageMaker.

### Personalized Recommendation Workflow

The backend checks `RecommendationCache` first. If a valid cache entry exists, results are returned directly without calling the endpoint. On a cache miss, the backend constructs the request context, calls SageMaker Endpoint, validates the response, writes to cache on a best-effort basis, and enriches results with movie metadata.

![Recommendation request flow through cache and SageMaker endpoint](/images/5-Workshop/5.1-Workshop-overview/backend-request-flow.jpg)

*Recommendation request flow: check the cache, invoke SageMaker on a cache miss, and retrieve metadata from the Movies table.*

## Retraining Workflow

1. Kaggle CSV files are profiled, cleaned, and MovieLens IDs are mapped to TMDB movie IDs.
2. The pipeline creates content features, interaction matrices, and temporal dataset splits.
3. The ALS model is trained and evaluated offline.
4. The promotion gate evaluates metrics and decides whether to update the `LATEST.json` pointer.
5. Model artifacts and evaluation reports are synchronized to S3.
6. Operational user interactions can be exported from DynamoDB to S3 for subsequent retraining runs.

## Service Roles Matrix

| Component | Role |
|---|---|
| Amazon S3 | Persistent storage for datasets, model artifacts, and evaluation reports |
| Amazon DynamoDB | Request-time storage for movie metadata, user accounts, interactions, and recommendation cache |
| SageMaker Processing Job | Executes batch model retraining jobs |
| SageMaker Endpoint | Target endpoint invoked by backend recommendation providers |
| Amazon EC2 | Hosts web application containers (React + FastAPI) and runs scheduled retraining |
| AWS IAM | Enforces least-privilege permission isolation for deployment, EC2 runtime, and SageMaker execution |
| Amazon VPC & Internet Gateway | Provides network boundary (Public Subnet) and internet routing for public HTTPS access and GitHub Actions SSH deployments |
| Amazon CloudWatch | Collects container application logs and SageMaker Processing Job execution metrics |
| AWS Budgets | Tracks project cloud spending against budget thresholds ($200 AWS credit guardrail) |

## Unimplemented Boundaries

{{% notice warning %}}
The repository currently lacks a serving handler or packaging script to deploy `RecommendationEngine` directly as a SageMaker endpoint. While the local recommendation engine and backend endpoint client interface can be tested independently, a new real-time endpoint cannot be provisioned solely from the current codebase.
{{% /notice %}}

The training path and request-time path form a closed deployment loop only after providing a serving handler, model bundle, and scripts to provision SageMaker Model, EndpointConfig, and Endpoint resources.

## Knowledge Check

- The guest path does not interact with SageMaker.
- The model returns only movie references, scores, and reason codes; metadata is resolved from the `Movies` table.
- A SageMaker Processing Job is distinct from a real-time endpoint deployment.
- The Interaction API records user behavior and does not directly trigger recommendations.

**Reference Sources:** `README.md`, `backend/app/container.py`, `backend/app/services/recommendation_service.py`, `backend/app/services/sagemaker_recommendation_provider.py`, and ML submodule at pinned commit.
