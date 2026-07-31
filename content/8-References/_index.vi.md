---
title: "Tài liệu tham khảo"
date: 2026-07-31
weight: 8
chapter: false
pre: " <b> 8. </b> "
---

Mục này tập hợp mã nguồn, bản demo, tài liệu AWS chính thức và các tệp được dùng để đối chiếu nội dung workshop hệ thống gợi ý phim.

## Tài nguyên dự án

| Tài nguyên | Liên kết |
|---|---|
| Mã nguồn hệ thống gợi ý phim | [GitHub - movie-recommendation-system](https://github.com/CaPPok/movie-recommendation-system) |
| Repository báo cáo và workshop | [GitHub - movie-project-report](https://github.com/tanghongai1704/movie-project-report) |
| Website báo cáo | [GitHub Pages - movie-project-report](https://tanghongai1704.github.io/movie-project-report/) |

## Demo

- [Google Drive - Demo hệ thống gợi ý phim](https://drive.google.com/drive/folders/1TNqHmVXZxYamXQ_ZqLBBzCpeKkqFaSAn?usp=sharing)

{{% notice note %}}
Hãy bảo đảm người xem có quyền truy cập thư mục Google Drive trước khi công bố báo cáo. Không đưa access token, secret key, mật khẩu hoặc URL tạm thời có chữ ký vào tài liệu.
{{% /notice %}}

## Tài liệu AWS chính thức

### Amazon S3 và DynamoDB

- [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [Tạo Amazon S3 bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html)
- [Amazon DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [Tạo DynamoDB table](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/getting-started-step-1.html)
- [Cấu hình TTL trong DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/time-to-live-ttl-how-to.html)
- [Point-in-time recovery trong DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery_Howitworks.html)

### Amazon SageMaker AI

- [Amazon SageMaker AI Developer Guide](https://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html)
- [SageMaker execution roles](https://docs.aws.amazon.com/sagemaker/latest/dg/sagemaker-roles.html)
- [CreateModel API](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateModel.html)
- [CreateEndpointConfig API](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateEndpointConfig.html)
- [Triển khai model cho real-time inference](https://docs.aws.amazon.com/sagemaker/latest/dg/realtime-endpoints-deploy-models.html)

### IAM và Amazon EC2

- [Các biện pháp bảo mật tốt nhất trong IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)

Các liên kết tài liệu AWS trên được kiểm tra lần cuối vào ngày **31/07/2026**.

## Nguồn đối chiếu workshop

| Đường dẫn trong repository mã nguồn | Mục đích đối chiếu |
|---|---|
| `backend/app/aws/infrastructure.py` | Định nghĩa cách backend truy cập tài nguyên AWS. |
| `docs/aws/dynamodb.md` | Schema và access pattern của DynamoDB. |
| `docs/aws/aws-setup.md` | Hướng dẫn cấu hình môi trường AWS. |
| `configs/data_pipeline.yaml` | Cấu hình pipeline xử lý dữ liệu. |
| `configs/model_serving.yaml` | Cấu hình model và cơ chế serving. |
| `configs/aws.yaml` | Region, S3 prefix, SageMaker và promotion criteria. |
| `scripts/sagemaker_retrain_job.py` | Khởi chạy hoặc dry-run SageMaker retraining job. |
| `scripts/test_sagemaker_endpoint.py` | Mô tả và kiểm thử SageMaker Endpoint. |

## Ghi chú cập nhật

- Ưu tiên permalink, release hoặc tag cố định để tài liệu không trỏ tới nội dung thay đổi ngoài ý muốn.
- Ghi lại commit SHA hoặc phiên bản phát hành được dùng làm bằng chứng cho workshop.
- Kiểm tra quyền truy cập của mã nguồn và thư mục demo trước khi phát hành.
- Kiểm tra lại toàn bộ liên kết trước mỗi lần cập nhật báo cáo.
