---
title: "Train, Evaluate, and Run a SageMaker Processing Job"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

## 1. Train the Collaborative Model

`train.py` reads the chronological training split, applies the positive, negative, and neutral rules, and then trains an **implicit ALS** model.

From the `ml` directory:

```bash
python train.py --version "<MODEL_VERSION>"
```

Versioned artifacts may include:

- User factors and item factors.
- User/item indexes.
- Manifest.
- Model configuration.
- Content artifact.

## 2. Evaluate the Model

```bash
python evaluate.py --version "<MODEL_VERSION>"
```

`evaluate.py` calculates these metric groups:

- HitRate.
- Precision.
- NDCG.
- Catalog coverage.

## 3. Promotion Gate

Run retraining in dry-run mode:

```bash
python retrain.py \
  --version "<MODEL_VERSION>" \
  --dry-run
```

`retrain.py` updates `LATEST.json` only when the candidate:

1. Has enough evaluated users.
2. Outperforms the popularity baseline on the configured metric.
3. Does not regress beyond the allowed limit compared with the currently served model.

A candidate that does not pass the gate is retained for investigation but is not promoted.

## 4. Run a SageMaker Processing Job

The launcher builds a source bundle and then uses `FrameworkProcessor` to run the `deploy/sagemaker_retrain.py` wrapper. The job retrieves input from S3 and writes output back to S3.

### 4.1. Prepare the Configuration

Before creating the job, confirm:

- Input is available at `s3://<S3_BUCKET_NAME>/<TRAINING_OR_EXPORT_PREFIX>/`.
- Output will be written to the approved `models/` and `evaluation/` prefixes.
- The SageMaker execution role can read the input, write the output, pull the correct container image from ECR, and write CloudWatch Logs.
- The principal creating the job has `iam:PassRole` only for the correct execution role.
- The instance type, instance count, EBS volume, and maximum runtime have been confirmed to control costs.

### 4.2. Configure the Processing Job in the AWS Console

1. Open **Amazon SageMaker AI** → **Processing jobs** → **Create processing job**.
2. Enter a **Processing job name** following the pattern `movie-rec-retrain-<TIMESTAMP>`.
3. Select `<SAGEMAKER_EXECUTION_ROLE_ARN>` under **IAM role**.
4. Under **Container**, enter `<PROCESSING_IMAGE_URI>` and the entrypoint/arguments corresponding to the retraining wrapper.
5. Under **Input**, specify the source S3 URI and local path inside the container, such as `/opt/ml/processing/input`.
6. Under **Output**, map a local output path, such as `/opt/ml/processing/output`, to the correct S3 model/evaluation prefix.
7. Select `<PROCESSING_INSTANCE_TYPE>`, instance count `1`, volume size, and a maximum runtime appropriate for the dataset.
8. Configure a VPC, subnet, and security group only when the workshop has private networking and suitable S3/ECR endpoints or NAT.
9. Add the tags `Project=movie-recommendation`, `Environment=<ENVIRONMENT>`, and `ModelVersion=<MODEL_VERSION>`.
10. Review all S3 URIs, the role, image, and cost, and then select **Create processing job**.

### 4.3. Launch with the Repository Launcher

After the dry run, review IAM and confirm the cost:

```bash
python scripts/sagemaker_retrain_job.py \
  --version "<MODEL_VERSION>" \
  --events "s3://<S3_BUCKET_NAME>/<INTERACTION_EXPORT_PREFIX>" \
  --wait
```

![Completed SageMaker Processing Job](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.2-train-and-deploy-model/sagemaker-processing-job.jpg)

*The retraining Processing Job completed after eight minutes.*

Check the status:

```bash
aws sagemaker describe-processing-job \
  --processing-job-name "<PROCESSING_JOB_NAME>" \
  --region "<AWS_REGION>"
```

The job must reach the `Completed` state.

## 5. Verify Artifacts and Reports

Confirm that S3 contains the correct model version, manifest, `LATEST.json`, and evaluation report.

## 6. Configure a SageMaker Real-Time Endpoint

- A serving handler compatible with the backend request/response contract.
- A process for creating `model.tar.gz`.
- A compatible image/runtime.
- Model, EndpointConfig, and Endpoint deployment.
- Rollback and autoscaling mechanisms.

### 6.1. Prepare the Model Artifact and Inference Image

1. Package the artifact in the `model.tar.gz` format supported by the inference container.
2. Upload the artifact to `s3://<S3_BUCKET_NAME>/models/<MODEL_VERSION>/model.tar.gz`.
3. Push the inference image to Amazon ECR or select a SageMaker-supported framework image.
4. Test the container locally with the request/response schema sent and received by the FastAPI provider.

### 6.2. Create a SageMaker Model

1. Open **Amazon SageMaker AI** → **Inference** → **Models** → **Create model**.
2. Enter a model name, such as `movie-rec-model-<MODEL_VERSION>`.
3. Select `<SAGEMAKER_EXECUTION_ROLE_ARN>`.
4. Select **Provide model artifacts and inference image location**.
5. Enter the inference image URI and the S3 URI of `model.tar.gz`.
6. Add the required environment variables, but do not place plaintext secrets in variables or tags.
7. Add tags and select **Create model**.

### 6.3. Create an Endpoint Configuration

1. Open **Inference** → **Endpoint configurations** → **Create endpoint configuration**.
2. Enter a name, such as `movie-rec-endpoint-config-<MODEL_VERSION>`.
3. Select the **Real-time** type.
4. Under **Production variants**, add the newly created model, set the variant name to `AllTraffic`, and set its initial weight to `1`.
5. Select `<SAGEMAKER_INFERENCE_INSTANCE_TYPE>` and initial instance count `1` for the workshop.
6. Optionally enable data capture to a separate S3 prefix after confirming that request/response data does not contain sensitive information outside policy.
7. Add tags and select **Create endpoint configuration**.

### 6.4. Create the Endpoint

1. Open **Inference** → **Endpoints** → **Create endpoint**.
2. Enter `<SAGEMAKER_ENDPOINT_NAME>` exactly as configured in `.env`.
3. Select **Use an existing endpoint configuration** and attach the configuration created above.
4. Select **Create endpoint**.
5. Wait for the status to change from `Creating` to `InService`. If it becomes `Failed`, inspect the failure reason and CloudWatch Logs before recreating it.
6. Run an endpoint smoke test with a payload that does not contain sensitive data, and then test the backend provider.

![SageMaker Endpoint in the InService state](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.2-train-and-deploy-model/sagemaker-endpoint.jpg)

*The existing endpoint is in the `InService` state.*
