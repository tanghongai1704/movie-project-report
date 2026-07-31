---
title: "Tài liệu tham khảo"
date: 2026-07-31
weight: 8
chapter: false
pre: " <b> 8. </b> "
---

Mục này tập trung các tài liệu, repository và tài nguyên hỗ trợ cho báo cáo và workshop hệ thống gợi ý phim.

## Tài nguyên dự án

| Tài nguyên | Trạng thái | Liên kết hoặc ghi chú |
|---|---|---|
| Repository báo cáo và workshop | Đã có | [GitHub - movie-project-report](https://github.com/tanghongai1704/movie-project-report) |
| Website báo cáo | Đã có | [GitHub Pages - movie-project-report](https://tanghongai1704.github.io/movie-project-report/) |
| Source code hệ thống gợi ý phim | Chưa cung cấp URL | Thay `<RECOMMENDATION_SOURCE_REPOSITORY_URL>` bằng liên kết repository thực tế. Nếu frontend, backend và ML nằm ở các repository khác nhau, hãy liệt kê từng repository. |
| Video demo | Chưa cung cấp URL | Thay `<DEMO_VIDEO_URL>` bằng liên kết YouTube, Google Drive hoặc nền tảng lưu trữ được phép truy cập. |

{{% notice note %}}
Trước khi công bố, hãy kiểm tra quyền truy cập của repository và video demo. Không đưa access token, secret key, mật khẩu hoặc URL tạm thời có chữ ký vào tài liệu.
{{% /notice %}}

## Tài liệu AWS chính thức

- [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [Amazon DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [Amazon SageMaker AI Developer Guide](https://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html)
- [Security best practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)

Các liên kết tài liệu AWS trên được kiểm tra lần cuối vào ngày **31/07/2026**.

## Tài liệu và source dùng để đối chiếu workshop

Các đường dẫn sau thuộc repository source của hệ thống gợi ý và cần được liên kết với URL source code khi repository được công bố:

- `backend/app/aws/infrastructure.py`: định nghĩa truy cập tài nguyên AWS từ backend.
- `docs/aws/dynamodb.md`: schema và access pattern DynamoDB.
- `docs/aws/aws-setup.md`: hướng dẫn cấu hình AWS.
- `configs/data_pipeline.yaml`: cấu hình xử lý dữ liệu.
- `configs/model_serving.yaml`: cấu hình model và serving.
- `configs/aws.yaml`: Region, S3 prefix, SageMaker và promotion criteria.
- `scripts/sagemaker_retrain_job.py`: khởi chạy hoặc dry-run retraining job.
- `scripts/test_sagemaker_endpoint.py`: mô tả và kiểm thử SageMaker Endpoint.

## Ghi chú cập nhật

- Thay các placeholder URL ngay khi repository source và video demo sẵn sàng.
- Ưu tiên permalink hoặc release/tag cố định để tài liệu không trỏ tới nội dung thay đổi ngoài ý muốn.
- Với source code dùng làm bằng chứng, nên ghi commit SHA hoặc release version đã được dùng để thực hiện workshop.
- Kiểm tra lại liên kết trước mỗi lần phát hành báo cáo.
