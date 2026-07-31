---
title: "Prepare the Recommendation Environment"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

## 1. Initialize the ML Submodule

From the repository root:

```bash
git submodule update --init --recursive
```

Confirm that the submodule contains:

- `configs/`
- `src/`
- `scripts/`
- `train.py`
- `evaluate.py`
- `retrain.py`

## 2. Create a Python Environment

From the `ml` directory:

```bash
python -m venv .venv
python -m pip install \
  -r requirements.txt \
  -r requirements-aws.txt
```

`requirements-aws.txt` adds boto3, botocore, and the SageMaker SDK required by the launcher.

## 3. Review the Configuration

| File | Contents |
|---|---|
| `configs/data_pipeline.yaml` | Data paths and pipeline steps |
| `configs/model_serving.yaml` | Hyperparameters and serving rules |
| `configs/aws.yaml` | Region, bucket, prefixes, processing instance, and promotion gate |

## 4. Prepare the Input

The pipeline must create:

- `interactions_train.parquet`.
- Validation and test splits.
- A content feature artifact.
- A serving lookup.
- An initial model directory or a specific model version.

Run validation before training:

```bash
python scripts/validate_data.py \
  --config configs/data_pipeline.yaml
```

Expected result: exit code `0`, and `interactions_train.parquet` contains user, movie, and interaction value columns.

## 5. Dry Run the SageMaker Processing Job

The dry run builds the source bundle and prints the job plan without calling AWS:

```bash
python scripts/sagemaker_retrain_job.py --dry-run
```

Verify that the output includes:

- Job name.
- AWS Region.
- S3 bucket and input prefix.
- Processing instance type.
- Execution role placeholder/configuration.
- Arguments passed to the wrapper.

The source bundle must not contain datasets or large artifacts.
