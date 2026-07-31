---
title: "Worklog Tuần 2"
date: 2026-06-08
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

{{% notice tip %}}
Tuần thứ hai tập trung tìm hiểu về các dịch vụ quản lý truy cập và lưu trữ trên AWS, đồng thời thực hành tạo và quản lý tài nguyên cơ bản nhằm làm quen với quy trình triển khai trên nền tảng đám mây.
{{% /notice %}}

## Mục tiêu tuần 2

- Tìm hiểu cơ chế quản lý người dùng và phân quyền trên AWS.
- Hiểu cách hoạt động của Amazon S3 và các trường hợp sử dụng phổ biến.
- Làm quen với Amazon EC2 thông qua các bài thực hành cơ bản.
- Nắm được quy trình tạo và quản lý tài nguyên trên AWS.

## Các công việc triển khai trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu AWS Identity and Access Management (IAM).<br>- Phân biệt Root User và IAM User.<br>- Tìm hiểu User, Group, Role và Policy. | 28/07/2026 | 28/07/2026 | https://docs.aws.amazon.com/iam/ |
| 3 | - Thực hành tạo IAM User.<br>- Gán Policy cho IAM User.<br>- Kiểm tra quyền truy cập trên AWS Console. | 29/07/2026 | 29/07/2026 | https://docs.aws.amazon.com/iam/latest/UserGuide/ |
| 4 | - Tìm hiểu Amazon S3.<br>- Tìm hiểu Bucket, Object, Storage Classes.<br>- Làm quen với cách quản lý dữ liệu trên S3. | 30/07/2026 | 30/07/2026 | https://docs.aws.amazon.com/s3/ |
| 5 | - Thực hành tạo S3 Bucket.<br>- Upload, Download và xóa Object.<br>- Kiểm tra quyền truy cập đối với Bucket. | 31/07/2026 | 31/07/2026 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/ |
| 6 | - Ôn tập EC2.<br>- Thực hành tạo EC2 Instance.<br>- Kết nối đến EC2 bằng SSH và làm quen với các thao tác quản lý cơ bản. | 01/08/2026 | 01/08/2026 | https://docs.aws.amazon.com/ec2/ |

## Kết quả đạt được tuần 2

Sau tuần thứ hai, đã hiểu rõ hơn về cách AWS quản lý quyền truy cập cũng như cách lưu trữ dữ liệu trên nền tảng đám mây.

Một số kết quả đạt được gồm:

- Hiểu được vai trò của AWS IAM trong việc quản lý người dùng và phân quyền.
- Phân biệt được:
  - Root User và IAM User.
  - User, Group và Role.
  - Managed Policy và Inline Policy.

- Thực hành tạo và quản lý IAM User.
- Gán Policy phù hợp để cấp quyền truy cập vào các dịch vụ AWS.

- Hiểu được cách hoạt động của Amazon S3 và các khái niệm:
  - Bucket
  - Object
  - Storage Class
  - Region

- Thực hiện thành công các thao tác cơ bản trên Amazon S3:
  - Tạo Bucket.
  - Upload và Download tệp.
  - Xóa Object.
  - Kiểm tra quyền truy cập Bucket.

- Ôn tập kiến thức về Amazon EC2 và thực hành:
  - Tạo EC2 Instance.
  - Cấu hình Security Group.
  - Tạo Key Pair.
  - Kết nối SSH đến EC2 Instance.

- Có thêm kinh nghiệm thao tác trên AWS Management Console và hiểu quy trình tạo, quản lý các tài nguyên cơ bản trên AWS.