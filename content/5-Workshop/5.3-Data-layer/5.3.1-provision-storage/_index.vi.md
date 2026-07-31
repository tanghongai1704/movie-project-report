---
title: "Kiểm tra và chuẩn bị tài nguyên lưu trữ"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

Repository không chứa Terraform, CloudFormation, CDK hoặc công cụ Infrastructure as Code tương đương. Phần dưới đây hướng dẫn tạo tài nguyên thủ công bằng AWS Console cho môi trường workshop, sau đó kiểm tra lại bằng Console và AWS CLI. Với production, các bước này cần được chuyển thành Infrastructure as Code và được platform/security team review.

## 1. Tạo năm bảng DynamoDB bằng AWS Console

### 1.1. Chọn Region và quy ước tên

1. Đăng nhập AWS Console bằng IAM user/role dành cho workshop, không dùng root user cho thao tác hằng ngày.
2. Chọn đúng `<AWS_REGION>` ở góc trên bên phải. Các tài nguyên S3, DynamoDB, SageMaker và EC2 của workshop nên dùng cùng Region; bằng chứng hiện tại sử dụng `ap-southeast-1`.
3. Dùng prefix theo môi trường, ví dụ `movie-rec-dev-`, rồi cập nhật đúng tên bảng vào `.env` của backend.

### 1.2. Tạo từng bảng

1. Mở **DynamoDB** → **Tables** → **Create table**.
2. Nhập table name, partition key và sort key theo bảng dưới đây. Tất cả key đều có kiểu **String**.
3. Trong **Table settings**, chọn **Customize settings**.
4. Chọn **On-demand** cho workshop có tải chưa ổn định. Chỉ dùng provisioned capacity khi đã có số liệu tải và kế hoạch capacity rõ ràng.
5. Giữ encryption mặc định của DynamoDB hoặc chọn customer managed KMS key nếu chính sách tổ chức yêu cầu.
6. Thêm tag tối thiểu: `Project=movie-recommendation`, `Environment=<ENVIRONMENT>` và `Owner=<OWNER>`.
7. Chọn **Create table**, chờ trạng thái chuyển sang `ACTIVE`, rồi lặp lại cho đủ năm bảng.

| Tên logic | Tên bảng gợi ý | Partition key | Sort key |
|---|---|---|---|
| Movies | `<ENV_PREFIX>-Movies` | `movie_id` (String) | Không có |
| PopularMovies | `<ENV_PREFIX>-PopularMovies` | `list_id` (String) | Không có |
| Users | `<ENV_PREFIX>-Users` | `user_id` (String) | Không có |
| UserInteractions | `<ENV_PREFIX>-UserInteractions` | `user_id` (String) | `interaction_key` (String) |
| RecommendationCache | `<ENV_PREFIX>-RecommendationCache` | `user_id` (String) | `scenario` (String) |


## 2. Kiểm tra năm bảng DynamoDB

Với từng tên bảng được cung cấp qua kênh bảo mật, chạy:

```bash
aws dynamodb describe-table \
  --table-name "<MOVIES_TABLE_NAME>" \
  --region "<AWS_REGION>"
```

Lặp lại cho bốn bảng còn lại và xác nhận:

| Bảng | HASH key | RANGE key |
|---|---|---|
| `Movies` | `movie_id` | Không có |
| `PopularMovies` | `list_id` | Không có |
| `Users` | `user_id` | Không có |
| `UserInteractions` | `user_id` | `interaction_key` |
| `RecommendationCache` | `user_id` | `scenario` |

Tất cả bảng phải ở trạng thái `ACTIVE`.

![Năm bảng DynamoDB ở trạng thái Active](/images/5-Workshop/5.3-Data-layer/5.3.1-provision-storage/dynamodb-tables.png)

*Năm bảng DynamoDB cùng partition key, sort key và trạng thái Active.*

## 3. Tạo S3 bucket bằng AWS Console

1. Mở **Amazon S3** → **General purpose buckets** → **Create bucket**.
2. Chọn **General purpose**, nhập `<S3_BUCKET_NAME>` duy nhất toàn cục và chọn đúng `<AWS_REGION>`.
3. Tại **Object Ownership**, giữ **Bucket owner enforced** để tắt ACL.
4. Tại **Block Public Access settings for this bucket**, giữ bật **Block all public access** và cả bốn tùy chọn con.
5. Tại **Bucket Versioning**, chọn **Enable**.
6. Thêm các tag `Project`, `Environment` và `Owner`.
7. Tại **Default encryption**, chọn **SSE-S3** cho workshop này. Nếu tổ chức yêu cầu kiểm soát khóa riêng, chọn SSE-KMS và bổ sung quyền KMS tương ứng.
8. Chọn **Create bucket**.
9. Mở bucket vừa tạo → **Create folder** để tạo các prefix logic: `raw/`, `processed/`, `training/`, `inference/`, `models/`, `evaluation/` và `interaction-exports/`.

## 4. Kiểm tra S3 bucket

![S3 bucket của hệ thống gợi ý phim](/images/5-Workshop/5.3-Data-layer/5.3.1-provision-storage/s3-bucket.png)

*S3 bucket được sử dụng để lưu dataset, model artifact và báo cáo.*

![Thông tin tổng quan của S3 bucket](/images/5-Workshop/5.3-Data-layer/5.3.1-provision-storage/s3-bucket-overview.png)

*Bucket được triển khai tại Region `ap-southeast-1`; ảnh tổng quan xác nhận trực tiếp ARN và Region của tài nguyên đang được kiểm tra.*

```bash
aws s3api head-bucket \
  --bucket "<S3_BUCKET_NAME>"

aws s3api get-public-access-block \
  --bucket "<S3_BUCKET_NAME>"

aws s3api get-bucket-encryption \
  --bucket "<S3_BUCKET_NAME>"

aws s3api get-bucket-versioning \
  --bucket "<S3_BUCKET_NAME>"
```

Bucket phải tồn tại, có Block Public Access và encryption phù hợp. Đối với bucket hiện tại, Block Public Access đang bật, mã hóa mặc định là SSE-S3 và versioning đang ở trạng thái `Enabled`. Source code ứng dụng không tự provision các thiết lập này; lifecycle vẫn cần được kiểm tra riêng nếu được áp dụng.

![Thiết lập Block Public Access của S3 bucket](/images/5-Workshop/5.3-Data-layer/5.3.1-provision-storage/s3-block-public-access.png)

*`Block all public access` đang ở trạng thái `On`, ngăn truy cập công khai qua ACL và bucket policy.*

![Thiết lập mã hóa mặc định của S3 bucket](/images/5-Workshop/5.3-Data-layer/5.3.1-provision-storage/s3-bucket-encryption.png)

*Mã hóa mặc định phía máy chủ sử dụng khóa do Amazon S3 quản lý (SSE-S3).*

![Thiết lập versioning của S3 bucket](/images/5-Workshop/5.3-Data-layer/5.3.1-provision-storage/s3-bucket-versioning.png)

*Bucket Versioning đang ở trạng thái `Enabled`; MFA delete đang `Disabled`.*

## 6. Kiểm tra các prefix

Chỉ lấy tối đa một object để tránh đọc dữ liệu không cần thiết:

```bash
aws s3api list-objects-v2 \
  --bucket "<S3_BUCKET_NAME>" \
  --prefix "<RAW_PREFIX>" \
  --max-items 1
```

Lặp lại với các prefix `processed`, `training`, `inference`, `models`, `evaluation` và `interaction exports`.

Prefix rỗng không nhất thiết là lỗi. `AccessDenied`, sai region hoặc bucket không tồn tại mới là dấu hiệu cần xử lý.

## 7. Khi tài nguyên chưa tồn tại

Nếu thiếu bảng hoặc bucket:

1. Dừng bước triển khai.
2. Ghi lại region, key schema, billing mode, TTL, encryption, lifecycle và IAM owner cần thiết.
3. Yêu cầu platform/security team cung cấp tài nguyên hoặc IaC đã được review.
