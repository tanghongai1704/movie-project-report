---
title: "Tổng kết và dọn dẹp tài nguyên"
date: 2026-07-30
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

Workshop đã trình bày:

- Data pipeline và S3 artifact layout.
- Năm bảng DynamoDB cùng access pattern.
- Huấn luyện ALS, evaluation và promotion gate.
- SageMaker Processing Job.
- FastAPI provider, cache và failure path.
- Giả định triển khai ứng dụng trên EC2.
- Ranh giới IAM theo nguyên tắc đặc quyền tối thiểu.

## Tài nguyên có thể phát sinh chi phí

- SageMaker real-time endpoint và endpoint instance.
- SageMaker Processing Job khi đang chạy.
- EC2 instance và attached storage.
- DynamoDB request, backup và table.
- S3 current object, non-current version và multipart upload.
- CloudWatch Logs.
- Data transfer.
- Các tài nguyên mạng tùy chọn chưa được mô tả trong repository.

## Thứ tự dọn tài nguyên

![Thứ tự phụ thuộc khi dọn tài nguyên AWS](/images/5-Workshop/5.6-Cleanup/cleanup-dependency-flow.jpg)

*Dọn tài nguyên theo thứ tự phụ thuộc để bảo toàn dữ liệu cần thiết và tránh resource còn tham chiếu lẫn nhau.*

Thực hiện theo thứ tự:

1. Dừng traffic, CI deployment và retraining scheduler.
2. Export hoặc lưu logs, báo cáo, model manifest và dữ liệu cần giữ.
3. Dừng SageMaker job đang hoạt động.
4. Xóa Endpoint trước EndpointConfig và Model nếu owner xác nhận các resource này thuộc workshop.
5. Stop hoặc terminate EC2 theo retention policy.
6. Export dữ liệu DynamoDB cần giữ rồi mới xóa test item hoặc table.
7. Xóa S3 current object, version và multipart upload trước khi xóa bucket.
8. Detach managed/inline policy trước khi xóa IAM role.
9. Kiểm tra Billing, Cost Explorer và resource inventory.


## Dữ liệu có thể mất vĩnh viễn

- User account và password hash.
- `UserInteractions` dùng cho retraining.
- `RecommendationCache`.
- Movie catalog và popular ranking.
- Raw/processed dataset.
- Model artifact, manifest và evaluation history.
- S3 object version.
- Log và audit evidence.

## Kiểm kê read-only

Các lệnh sau chỉ liệt kê tài nguyên, không thực hiện cleanup:

```bash
aws sagemaker list-endpoints \
  --region "<AWS_REGION>"

aws sagemaker list-processing-jobs \
  --region "<AWS_REGION>"

aws ec2 describe-instances \
  --region "<AWS_REGION>" \
  --filters Name=instance-state-name,Values=running

aws dynamodb list-tables \
  --region "<AWS_REGION>"

aws s3api list-objects-v2 \
  --bucket "<S3_BUCKET_NAME>" \
  --max-items 10
```