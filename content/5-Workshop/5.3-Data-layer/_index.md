---
title: "Data Layer with S3 and DynamoDB"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

Amazon S3 and Amazon DynamoDB serve two different roles:

- **S3** stores large datasets and artifacts used in batch workflows.
- **DynamoDB** serves lookups and stores state in the application's request path.

![Data processing flow from Kaggle CSV files to S3 and DynamoDB](/images/5-Workshop/5.3-Data-layer/data-ingestion-flow.jpg)

*Local data processing flow, artifact creation, and synchronization to S3. The DynamoDB loader remains an unimplemented component in the repository.*

## S3 Data Structure

The logical zones are:

- `raw`: initial CSV data.
- `processed`: cleaned data and features.
- `training`: training, validation, and test datasets.
- `inference`: lookup artifacts for the ML engine.
- `models`: version-managed model artifacts.
- `evaluation`: evaluation and promotion reports.
- `interaction exports`: feedback from DynamoDB for retraining.

The backend HTTP handler does not load datasets from S3 in the request path.

## Five DynamoDB Tables

| Table | Primary key | Purpose |
|---|---|---|
| `Movies` | `movie_id` | Movie metadata |
| `PopularMovies` | `list_id` | Popular movie lists for guests |
| `Users` | `user_id` | Accounts, profiles, and onboarding |
| `UserInteractions` | `user_id`, `interaction_key` | Clicks, watches, ratings, reactions, and shares |
| `RecommendationCache` | `user_id`, `scenario` | Context-specific recommendation cache |

`Movies` is the primary metadata source. `PopularMovies` and `RecommendationCache` store only movie references instead of duplicating all metadata.

## Relationship to the Recommendation System

1. Guest ranking reads `PopularMovies` and enriches the result with metadata from `Movies`.
2. User behavior is written to `UserInteractions`.
3. Interactions can be exported to S3 for the next retraining run.
4. The personalized cache stores `movie_id`, `score`, and `reason_code`.
5. Results from the cache or provider are enriched with `Movies` before the API response is returned.

## Next Steps

1. [Verify and prepare storage resources](5.3.1-provision-storage/)
2. [Process, load, and validate data](5.3.2-load-and-validate-data/)
