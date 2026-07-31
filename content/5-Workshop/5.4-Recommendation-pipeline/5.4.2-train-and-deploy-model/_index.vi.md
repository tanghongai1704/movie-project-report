---
title: "Huấn luyện, đánh giá và chạy SageMaker Processing Job"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

## 1. Huấn luyện collaborative model

`train.py` đọc chronological training split, áp dụng quy tắc positive, negative và neutral, sau đó huấn luyện mô hình **implicit ALS**.

Tại thư mục `ml`:

```bash
python train.py --version "<MODEL_VERSION>"
```

Artifact được lưu theo version và có thể gồm:

- User factors và item factors.
- User/item index.
- Manifest.
- Model configuration.
- Content artifact.

## 2. Đánh giá mô hình

```bash
python evaluate.py --version "<MODEL_VERSION>"
```

`evaluate.py` tính các nhóm metric:

- HitRate.
- Precision.
- NDCG.
- Catalog coverage.

{{% notice note %}}
Không sử dụng số metric từ tài liệu hoặc báo cáo cũ làm kết quả hiện tại. Report phải lấy metric từ artifact được tạo bởi chính lần chạy workshop.
{{% /notice %}}

## 3. Promotion gate

Chạy retraining ở chế độ dry-run:

```bash
python retrain.py \
  --version "<MODEL_VERSION>" \
  --dry-run
```

`retrain.py` chỉ cập nhật `LATEST.json` khi candidate:

1. Có đủ số user được đánh giá.
2. Tốt hơn popularity baseline theo metric cấu hình.
3. Không regression quá giới hạn cho phép so với model đang phục vụ.

Candidate không vượt gate vẫn được giữ lại để điều tra nhưng không được promote.

{{% notice warning %}}
`--force-promote` bỏ qua quality gate và không nên được sử dụng trong quy trình workshop thông thường.
{{% /notice %}}

## 4. Chạy SageMaker Processing Job

Launcher dựng source bundle rồi dùng `FrameworkProcessor` để chạy wrapper `deploy/sagemaker_retrain.py`. Job lấy input từ S3 và đẩy output trở lại S3.

### 4.1. Chuẩn bị cấu hình

Trước khi tạo job, xác nhận:

- Input đã có tại `s3://<S3_BUCKET_NAME>/<TRAINING_OR_EXPORT_PREFIX>/`.
- Output sẽ ghi vào prefix `models/` và `evaluation/` đã được phê duyệt.
- SageMaker execution role có quyền đọc input, ghi output, kéo đúng container image từ ECR và ghi CloudWatch Logs.
- Principal tạo job chỉ được `iam:PassRole` đối với đúng execution role.
- Instance type, số lượng instance, EBS volume và maximum runtime đã được xác nhận để kiểm soát chi phí.

### 4.2. Cấu hình Processing Job trên AWS Console

1. Mở **Amazon SageMaker AI** → **Processing jobs** → **Create processing job**.
2. Nhập **Processing job name** theo mẫu `movie-rec-retrain-<TIMESTAMP>`.
3. Chọn `<SAGEMAKER_EXECUTION_ROLE_ARN>` tại **IAM role**.
4. Tại **Container**, nhập `<PROCESSING_IMAGE_URI>` và entrypoint/arguments tương ứng với wrapper retraining.
5. Tại **Input**, khai báo S3 URI nguồn và local path trong container, ví dụ `/opt/ml/processing/input`.
6. Tại **Output**, ánh xạ local output path, ví dụ `/opt/ml/processing/output`, tới đúng S3 model/evaluation prefix.
7. Chọn `<PROCESSING_INSTANCE_TYPE>`, instance count `1`, volume size và maximum runtime phù hợp với dataset.
8. Chỉ cấu hình VPC, subnet và security group khi workshop đã có private networking và S3/ECR endpoint hoặc NAT phù hợp.
9. Thêm tag `Project=movie-recommendation`, `Environment=<ENVIRONMENT>` và `ModelVersion=<MODEL_VERSION>`.
10. Review toàn bộ S3 URI, role, image và chi phí rồi chọn **Create processing job**.

<!-- IMAGE-5.4.2-PROCESSING-01: Màn hình cấu hình Processing Job gồm role, container, input, output và compute resources. -->

### 4.3. Khởi chạy bằng launcher của repository

Sau khi dry-run, review IAM và xác nhận chi phí:

```bash
python scripts/sagemaker_retrain_job.py \
  --version "<MODEL_VERSION>" \
  --events "s3://<S3_BUCKET_NAME>/<INTERACTION_EXPORT_PREFIX>" \
  --wait
```

![SageMaker Processing Job hoàn tất](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.2-train-and-deploy-model/sagemaker-processing-job.jpg)

*Processing Job phục vụ retraining đã hoàn tất sau tám phút.*

<!-- IMAGE-5.4.2-PROCESSING-02: Trang chi tiết Processing Job ở trạng thái Completed và liên kết CloudWatch Logs. -->

Kiểm tra trạng thái:

```bash
aws sagemaker describe-processing-job \
  --processing-job-name "<PROCESSING_JOB_NAME>" \
  --region "<AWS_REGION>"
```

Job phải đạt trạng thái `Completed`.

## 5. Kiểm tra artifact và báo cáo

Xác nhận S3 có đúng model version, manifest, `LATEST.json` và evaluation report.

<!-- IMAGE-5.4.2-02: Evaluation report và promotion decision, không chứa dữ liệu người dùng. -->

## 6. Cấu hình SageMaker real-time Endpoint

{{% notice warning %}}
Repository không chứa `model_fn`, `input_fn`, `predict_fn`, `output_fn`, `transform_fn` hoặc script tạo SageMaker Model, EndpointConfig và Endpoint. Vì vậy không có câu lệnh hợp lệ để hoàn tất endpoint deployment từ source hiện tại.
{{% /notice %}}

Chỉ thực hiện các bước dưới đây sau khi team bổ sung:

- Serving handler tương thích request/response contract của backend.
- Quy trình tạo `model.tar.gz`.
- Image/runtime tương thích.
- Model, EndpointConfig và Endpoint deployment.
- Cơ chế rollback và autoscaling.

### 6.1. Chuẩn bị model artifact và inference image

1. Đóng gói artifact đúng định dạng `model.tar.gz` mà inference container hỗ trợ.
2. Upload artifact tới `s3://<S3_BUCKET_NAME>/models/<MODEL_VERSION>/model.tar.gz`.
3. Push inference image tới Amazon ECR hoặc chọn image framework được SageMaker hỗ trợ.
4. Kiểm thử container cục bộ với request/response schema mà FastAPI provider đang gửi và nhận.

### 6.2. Tạo SageMaker Model

1. Mở **Amazon SageMaker AI** → **Inference** → **Models** → **Create model**.
2. Nhập model name, ví dụ `movie-rec-model-<MODEL_VERSION>`.
3. Chọn `<SAGEMAKER_EXECUTION_ROLE_ARN>`.
4. Chọn **Provide model artifacts and inference image location**.
5. Nhập inference image URI và S3 URI của `model.tar.gz`.
6. Thêm environment variables cần thiết nhưng không đưa secret dạng plain text vào biến hoặc tag.
7. Thêm tag rồi chọn **Create model**.

<!-- IMAGE-5.4.2-MODEL-01: Trang chi tiết SageMaker Model với model artifact, inference image và execution role. -->

### 6.3. Tạo Endpoint configuration

1. Vào **Inference** → **Endpoint configurations** → **Create endpoint configuration**.
2. Nhập tên, ví dụ `movie-rec-endpoint-config-<MODEL_VERSION>`.
3. Chọn loại **Real-time**.
4. Trong **Production variants**, thêm model vừa tạo, đặt variant name `AllTraffic` và initial weight `1`.
5. Chọn `<SAGEMAKER_INFERENCE_INSTANCE_TYPE>` và initial instance count `1` cho workshop.
6. Tùy chọn bật data capture tới prefix S3 riêng sau khi xác nhận dữ liệu request/response không chứa thông tin nhạy cảm ngoài chính sách.
7. Thêm tag và chọn **Create endpoint configuration**.

<!-- IMAGE-5.4.2-ENDPOINT-01: Endpoint configuration thể hiện model, production variant, instance type và instance count. -->

### 6.4. Tạo Endpoint

1. Vào **Inference** → **Endpoints** → **Create endpoint**.
2. Nhập `<SAGEMAKER_ENDPOINT_NAME>` trùng với giá trị cấu hình trong `.env`.
3. Chọn **Use an existing endpoint configuration** và gắn configuration vừa tạo.
4. Chọn **Create endpoint**.
5. Chờ trạng thái chuyển từ `Creating` sang `InService`. Nếu `Failed`, mở failure reason và CloudWatch Logs trước khi tạo lại.
6. Chạy endpoint smoke test bằng payload không chứa dữ liệu nhạy cảm, sau đó kiểm tra backend provider.

![SageMaker endpoint ở trạng thái InService](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.2-train-and-deploy-model/sagemaker-endpoint.jpg)

*Endpoint hiện hữu đang ở trạng thái `InService`. Ảnh xác nhận tài nguyên đang hoạt động nhưng không thay thế phần serving/deployment source còn thiếu trong repository.*

<!-- IMAGE-5.4.2-ENDPOINT-02: Trang endpoint ở trạng thái InService và CloudWatch invocation metrics. -->

**Tài liệu AWS chính thức:** [SageMaker execution roles](https://docs.aws.amazon.com/sagemaker/latest/dg/sagemaker-roles.html), [CreateModel API](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateModel.html), [CreateEndpointConfig API](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateEndpointConfig.html) và [triển khai model cho real-time inference](https://docs.aws.amazon.com/sagemaker/latest/dg/realtime-endpoints-deploy-models.html).

## Tiêu chí hoàn tất

- [ ] Model version directory được tạo.
- [ ] Evaluation report có metric name và số user được đánh giá.
- [ ] Promotion report ghi rõ pass/fail cho từng gate.
- [ ] Processing Job đạt `Completed`.
- [ ] S3 có artifact và report đúng prefix.
- [ ] Endpoint chỉ được đánh dấu hoàn chỉnh khi serving package, Model, EndpointConfig, Endpoint và smoke test đều có bằng chứng tương ứng.

**Nguồn đối chiếu:** `ml/train.py`, `ml/evaluate.py`, `ml/retrain.py` và `ml/scripts/sagemaker_retrain_job.py`.
