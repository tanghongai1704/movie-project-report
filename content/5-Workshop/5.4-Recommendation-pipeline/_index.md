---
title: "Recommendation Pipeline"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

The pipeline supports multiple strategies for different user states.

## Recommendation Strategies

- **Popularity:** IMDb-style weighted popularity ranking for unauthenticated guests.
- **Content-based:** TF-IDF and cosine similarity for new users based on onboarding genres.
- **Collaborative filtering:** implicit ALS to generate candidates for returning users.
- **Hybrid ranking:** combines weighted Reciprocal Rank Fusion, recent similarity, and business-rule filtering.

![Model training, evaluation, artifact management, and serving flow](/images/5-Workshop/5.4-Recommendation-pipeline/recommendation-pipeline-flow.png)

*The pipeline trains and evaluates the model, checks the promotion gate, stores artifacts in S3, and then serves the model through a SageMaker Endpoint and FastAPI provider.*

## Input and Output

Pipeline input includes:

- Movie features.
- Time-based interaction splits.
- Onboarding genres.
- Recent interactions.
- A list of movie IDs to exclude.

The recommendation engine returns:

- An ordered list of `movie_id` values.
- `score`.
- `reason_code`.
- `reason_context`.

The backend enriches the results with metadata from the `Movies` table and then caches `movie_id`, `score`, and `reason_code`.

## Role of SageMaker and EC2

The SageMaker Processing Job is implemented to run retraining. EC2 can run the Docker application or, optionally, run periodic retraining with systemd.

The backend already contains a client that invokes a SageMaker real-time endpoint, but the source currently does not include endpoint packaging and deployment.

## Contents

1. [Prepare the recommendation environment](5.4.1-prepare-environment/)
2. [Train, evaluate, and run a SageMaker Processing Job](5.4.2-train-and-deploy-model/)
3. [Integrate the application on EC2](5.4.3-integrate-ec2-application/)
4. [End-to-end testing](5.4.4-end-to-end-testing/)
5. [Model testing](5.4.5-model-testing/)
