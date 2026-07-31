---
title: "Worklog Tuần 2"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

{{% notice tip %}}
Tuần thứ hai tập trung tìm hiểu các dịch vụ lưu trữ dữ liệu trên AWS, đặc biệt là Amazon S3 và Amazon DynamoDB. Đồng thời thực hành tạo các tài nguyên đầu tiên và kết nối ứng dụng với AWS thông qua AWS SDK.
{{% /notice %}}

## Mục tiêu tuần 2

- Tìm hiểu Amazon S3 và Amazon DynamoDB.
- Hiểu cách lưu trữ dữ liệu trên AWS.
- Thực hành tạo S3 Bucket và DynamoDB Table.
- Làm quen với AWS SDK (Boto3).
- Chuẩn bị dữ liệu phục vụ project.

## Các công việc triển khai trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu Amazon S3.<br>- Ôn tập Bucket, Object và Storage Class.<br>- Tìm hiểu các trường hợp sử dụng của S3. | 22/06/2026 | 22/06/2026 | https://docs.aws.amazon.com/s3/ |
| 3 | - Tìm hiểu Amazon DynamoDB.<br>- Hiểu Table, Item và Attribute.<br>- Tìm hiểu Partition Key và Sort Key. | 23/06/2026 | 23/06/2026 | https://docs.aws.amazon.com/dynamodb/ |
| 4 | - Thực hành tạo Amazon S3 Bucket.<br>- Tạo DynamoDB Table.<br>- Cấu hình IAM để cấp quyền truy cập. | 24/06/2026 | 24/06/2026 | AWS Documentation |
| 5 | - Tìm hiểu AWS SDK (Python - Boto3).<br>- Kết nối ứng dụng với S3 và DynamoDB.<br>- Thử đọc và ghi dữ liệu. | 25/06/2026 | 25/06/2026 | https://boto3.amazonaws.com/ |
| 6 | - Upload dữ liệu mẫu lên Amazon S3.<br>- Kiểm tra kết nối giữa ứng dụng và AWS.<br>- Ôn tập các kiến thức đã học. | 26/06/2026 | 26/06/2026 | AWS Documentation |

## Kết quả đạt được tuần 2

Sau tuần thứ hai, đã làm quen với các dịch vụ lưu trữ dữ liệu phổ biến trên AWS và hiểu cách ứng dụng có thể tương tác với các dịch vụ này.

Một số kết quả đạt được gồm:

- Hiểu được vai trò của Amazon S3 trong việc lưu trữ dữ liệu và tệp tin.
- Hiểu được cấu trúc dữ liệu của Amazon DynamoDB theo mô hình NoSQL.
- Thực hành tạo thành công:
  - Amazon S3 Bucket.
  - Amazon DynamoDB Table.
  - IAM Policy để cấp quyền truy cập.

- Sử dụng AWS SDK (Boto3) để:
  - Kết nối với Amazon S3.
  - Kết nối với Amazon DynamoDB.
  - Thực hiện các thao tác đọc và ghi dữ liệu cơ bản.

- Upload thành công dữ liệu mẫu lên Amazon S3 phục vụ quá trình phát triển project.

- Hiểu quy trình xác thực và kết nối ứng dụng với các dịch vụ AWS thông qua IAM User và Access Key.