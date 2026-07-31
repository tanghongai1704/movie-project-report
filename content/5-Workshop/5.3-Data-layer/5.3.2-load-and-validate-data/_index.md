---
title: "Process, Load, and Validate Data"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

This section runs the data pipeline locally, verifies deterministic output, synchronizes artifacts to S3, and clearly identifies the current gap in loading data into DynamoDB.

## 1. Prepare Raw Data

Place the correctly named CSV files in the raw directory declared by `ml/configs/data_pipeline.yaml`. `ratings.csv` and `links.csv` are the primary historical interaction sources; the `small` datasets are used only for profiling.

Do not commit datasets to Git.

## 2. Run the Data Pipeline

From the `ml` directory:

```bash
python scripts/run_data_pipeline.py \
  --config configs/data_pipeline.yaml
```

The pipeline performs these steps in sequence:

1. Profile the data.
2. Clean the data.
3. Build features.
4. Create splits and serving exports.
5. Run validation.

A critical validation failure must return a nonzero exit code.

## 3. Data Processing Rules

- MovieLens movie IDs are mapped to TMDB IDs using `links.csv`.
- For duplicate metadata, the more complete record is retained according to a deterministic rule.
- For duplicate interaction aliases, the latest timestamp is retained.
- Data is split per user: the most recent interaction goes to the test set, and the preceding interaction goes to the validation set.
- JSONL output contains only serving fields on the allowlist.

Example schema for illustration only, not production data:

```json
{
  "movie_id": "<MOVIE_ID>",
  "title": "<TITLE>",
  "genres": ["<GENRE>"],
  "poster_path": "<RELATIVE_TMDB_PATH>"
}
```

## 4. Verify Determinism and Run Tests

```bash
python scripts/check_determinism.py \
  --config configs/data_pipeline.yaml

python -m pytest -q
```

Expected results:

- Validation has no critical failures.
- The determinism check succeeds.
- The ML tests pass in a compatible dependency environment.

## 5. Synchronize to S3

Always run a dry run first:

```bash
python scripts/aws_sync.py push --dry-run
```

After reviewing the object list and receiving approval:

```bash
python scripts/aws_sync.py push
```

![Data prefixes in the S3 bucket](/images/5-Workshop/5.3-Data-layer/5.3.2-load-and-validate-data/s3-bucket-prefixes.png)

*The `datasets`, `evaluation`, `inference`, `logs`, `models`, and `training` data zones in the S3 bucket.*

## 6. Load Data into DynamoDB

The pipeline creates serving JSONL files, but the repository does not yet contain an official loader for `Movies` and `PopularMovies`.

## 7. Export Interactions for Retraining

The exporter scans the `UserInteractions` table, normalizes reaction/rating state, and can write JSONL locally or upload it to the interaction export prefix:

```bash
python scripts/export_interactions.py --upload
```

![Production feedback flow from the frontend to retraining](/images/5-Workshop/5.3-Data-layer/5.3.2-load-and-validate-data/production-feedback-flow.jpg)

*Interactions are written to DynamoDB, exported to S3, and then used as input for the next retraining run.*
