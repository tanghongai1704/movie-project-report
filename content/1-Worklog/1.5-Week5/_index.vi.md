---
title: "Worklog Tuần 5"
date: 2026-06-29
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

{{% notice tip %}}
Tuần thứ năm tập trung tìm hiểu các dịch vụ lưu trữ dữ liệu trên AWS, đặc biệt là Amazon S3 và Amazon DynamoDB. Đồng thời thực hành tạo tài nguyên và kết nối ứng dụng với các dịch vụ này để chuẩn bị cho việc xây dựng project.
{{% /notice %}}

## Mục tiêu tuần 5

- Tìm hiểu Amazon S3 và Amazon DynamoDB.
- Hiểu cách lưu trữ dữ liệu trên AWS.
- Thực hành tạo Bucket và DynamoDB Table.
- Kết nối ứng dụng với các dịch vụ AWS thông qua AWS SDK.
- Chuẩn bị dữ liệu cho project.

## Các công việc triển khai trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu Amazon S3.<br>- Ôn tập Bucket, Object và Storage Class.<br>- Tìm hiểu các trường hợp sử dụng của S3 trong thực tế. | 18/08/2026 | 18/08/2026 | https://docs.aws.amazon.com/s3/ |
| 3 | - Tìm hiểu Amazon DynamoDB.<br>- Hiểu các khái niệm Table, Item và Attribute.<br>- Tìm hiểu Partition Key và Sort Key. | 19/08/2026 | 19/08/2026 | https://docs.aws.amazon.com/dynamodb/ |
| 4 | - Thực hành tạo S3 Bucket.<br>- Tạo DynamoDB Table.<br>- Cấu hình IAM để ứng dụng có quyền truy cập các dịch vụ. | 20/08/2026 | 20/08/2026 | AWS Documentation |
| 5 | - Tìm hiểu AWS SDK (Python - Boto3).<br>- Thực hành kết nối ứng dụng với S3 và DynamoDB.<br>- Thử đọc và ghi dữ liệu. | 21/08/2026 | 21/08/2026 | https://boto3.amazonaws.com/ |
| 6 | - Chuẩn bị dữ liệu cho project.<br>- Upload dữ liệu mẫu lên Amazon S3.<br>- Kiểm tra kết nối giữa ứng dụng và các dịch vụ AWS. | 22/08/2026 | 22/08/2026 | AWS Documentation |

## Kết quả đạt được tuần 5

Sau tuần thứ năm, đã làm quen với các dịch vụ lưu trữ dữ liệu phổ biến trên AWS và có thể kết nối ứng dụng với các dịch vụ này.

Một số kết quả đạt được gồm:

- Hiểu được vai trò của Amazon S3 trong việc lưu trữ dữ liệu và tệp tin.
- Hiểu được cấu trúc của Amazon DynamoDB và cách tổ chức dữ liệu theo NoSQL.
- Thực hành tạo thành công:
  - Amazon S3 Bucket.
  - Amazon DynamoDB Table.
  - IAM Policy để cấp quyền truy cập.

- Sử dụng AWS SDK (Boto3) để:
  - Kết nối với Amazon S3.
  - Kết nối với Amazon DynamoDB.
  - Thực hiện các thao tác đọc và ghi dữ liệu cơ bản.

- Upload thành công dữ liệu mẫu lên Amazon S3 phục vụ cho quá trình phát triển project.

- Hiểu quy trình kết nối ứng dụng với các dịch vụ AWS thông qua Access Key và IAM User.

- Chuẩn bị được môi trường lưu trữ dữ liệu để phục vụ cho các chức năng sẽ được phát triển ở những tuần tiếp theo.