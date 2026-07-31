---
title: "Data Layer with S3 and DynamoDB"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

Amazon S3 and Amazon DynamoDB serve two distinct roles in the architecture:

- **S3** stores large-scale datasets and model artifacts used primarily in batch processes.
- **DynamoDB** handles item lookups and maintains state within the application's request-time path.

{{< mermaid align="center" >}}
flowchart LR
    Raw[Kaggle CSV] --> Pipeline[Data Pipeline]
    Pipeline --> Local[Parquet, JSONL, Artifacts]
    Local --> S3[(Amazon S3)]
    Local -. No loader script .-> Catalog[(Movies / PopularMovies)]
    UI[Frontend] --> API[FastAPI]
    API --> Interactions[(UserInteractions)]
    Interactions --> Exporter[Interaction Exporter]
    Exporter --> S3
    S3 --> Training[Retraining]
{{< /mermaid >}}

![Data processing flow from Kaggle CSV to S3 and DynamoDB](/images/5-Workshop/5.3-Data-layer/data-ingestion-flow.jpg)

*Local data processing and S3 synchronization flow. The DynamoDB loader remains missing from the repository.*

## S3 Data Structure

Logical storage zones include:

- `raw`: Initial raw CSV datasets.
- `processed`: Cleaned datasets and engineered feature artifacts.
- `training`: Temporal splits for train, validation, and test sets.
- `inference`: Lookup artifacts for the ML engine.
- `models`: Version-controlled model artifacts.
- `evaluation`: Performance evaluation reports and promotion logs.
- `interaction exports`: Exported user feedback from DynamoDB for retraining.

The backend HTTP request handlers do not load datasets directly from S3 during API execution.

## Five DynamoDB Tables

| Table | Primary Key | Purpose |
|---|---|---|
| `Movies` | `movie_id` | Master movie metadata catalog |
| `PopularMovies` | `list_id` | Popular movie rankings for guest users |
| `Users` | `user_id` | User accounts, profiles, and onboarding status |
| `UserInteractions` | `user_id`, `interaction_key` | User actions (clicks, watches, ratings, reactions, shares) |
| `RecommendationCache` | `user_id`, `scenario` | Contextual recommendation cache |

`Movies` is the single source of truth for movie metadata. `PopularMovies` and `RecommendationCache` store only movie ID references rather than duplicating full metadata payload records.

## Integration with Recommendation System

1. Guest ranking fetches `PopularMovies` references and enriches them with `Movies` metadata.
2. User interactions are written to `UserInteractions`.
3. Interaction logs can be periodically exported to S3 for future model retraining.
4. Personalized recommendation cache stores `movie_id`, `score`, and `reason_code`.
5. Recommendation results (from cache or SageMaker provider) are enriched using `Movies` before returning responses to the API caller.



## Next Steps

1. [Verify and Prepare Storage Resources](5.3.1-provision-storage/)
2. [Process, Load, and Validate Data](5.3.2-load-and-validate-data/)

**Reference Sources:** `docs/aws/s3.md`, `docs/aws/dynamodb.md`, `backend/app/container.py`, and ML data pipeline configs.
