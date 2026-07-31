---
title: "Prerequisites"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## 1. Required Tools

The workshop environment requires:

- Git with submodule support.
- Docker Engine and Docker Compose v2.
- AWS CLI v2.
- Python 3.11.
- Node.js 20 and npm.
- Internet access to install dependencies.

The backend container uses Python 3.11; the frontend Dockerfile uses Node.js 20.

## 2. Initialize the Repository and ML Submodule

The ML project is managed as a Git submodule. From the repository root, run:

```bash
git submodule update --init --recursive
```

After it completes, verify the following file:

```bash
test -f ml/train.py
```

The command must return exit code `0`.

## 3. Verify the AWS Identity

Prefer **IAM Identity Center/SSO** for developers and IAM roles for EC2/SageMaker. Do not store long-lived access keys in Git.

```bash
aws sts get-caller-identity --region "<AWS_REGION>"
```
![AWS identity](/images/5-Workshop/5.3-Step-by-step/aws-identity.jpg)

## 4. Required Existing Resources

The repository does not provision infrastructure automatically. Before starting the workshop, you need:

- An S3 bucket with the logical prefixes defined in `.env.example`.
- Five DynamoDB tables with the correct key schemas.
- A valid `PopularMovies list_id`.
- A compatible SageMaker endpoint in the `InService` state when testing a personalized cache miss.
- A provisioned EC2 instance when using the GitHub deployment workflow.

## 5. Prepare the Dataset

The CSV files are declared in `ml/configs/data_pipeline.yaml` and include metadata, ratings, links, credits, keywords, and smaller profiling datasets. Raw data is not committed to Git; place these files in the input directory configured in the ML project.

## 6. Configure Environment Variables

From the repository root:

```bash
cp .env.example .env
```

Set the values for the target environment:

- `<AWS_REGION>`
- `<JWT_SECRET_VALUE>`
- Names of the five DynamoDB tables
- `<POPULAR_LIST_ID>`
- `<S3_BUCKET_NAME>` and the prefixes
- `<SAGEMAKER_ENDPOINT_NAME>`
- Frontend API URL and TMDB poster URL

Then validate Docker Compose:

```bash
docker compose config --quiet
```

Expected result: exit code `0`, and `.env` does not appear in `git status`.

## 7. Install Dependencies

```bash
cd backend
python -m pip install -r requirements.txt

cd ../frontend
npm ci

cd ../ml
python -m pip install -r requirements.txt -r requirements-aws.txt
```

## 8. Minimum-Permission Matrix

| Principal | Required permissions |
|---|---|
| Backend runtime | STS identity; DynamoDB describe/read/write/query/scan according to the access patterns; S3 list for startup validation; SageMaker describe/invoke |
| S3/ML tooling | List/Get/Put on approved prefixes |
| SageMaker launcher | Create and describe Processing Jobs; `iam:PassRole` for the correct execution role |
| SageMaker execution role | Read/write the required S3 prefixes and write CloudWatch Logs |
| EC2 application | Use an instance profile instead of static credentials |
