---
title: "Worklog Tuần 7"
date: 2026-07-27
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

{{% notice tip %}}
Tuần thứ bảy tập trung triển khai hệ thống lên Amazon EC2, cấu hình Docker Compose trên máy chủ và thiết lập quy trình triển khai tự động thông qua GitHub Actions.
{{% /notice %}}

## Mục tiêu tuần 7

- Triển khai ứng dụng lên Amazon EC2.
- Cấu hình Docker và Docker Compose trên máy chủ.
- Thiết lập GitHub Actions.
- Kết nối ứng dụng với các dịch vụ AWS.
- Kiểm thử hệ thống sau khi triển khai.

## Các công việc triển khai trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Chuẩn bị môi trường triển khai trên EC2.<br>- Clone source code từ GitHub.<br>- Kiểm tra môi trường chạy. | 27/07/2026 | 27/07/2026 | AWS EC2 Documentation |
| 3 | - Cài đặt Docker và Docker Compose trên EC2.<br>- Build frontend và backend.<br>- Chạy ứng dụng bằng Docker Compose. | 28/07/2026 | 28/07/2026 | Docker Documentation |
| 4 | - Thiết lập GitHub Actions.<br>- Cấu hình SSH Deploy Key.<br>- Tự động cập nhật source code lên EC2. | 29/07/2026 | 29/07/2026 | GitHub Actions Documentation |
| 5 | - Cấu hình biến môi trường.<br>- Kiểm tra kết nối với S3, DynamoDB và SageMaker.<br>- Khắc phục lỗi phát sinh khi deploy. | 30/07/2026 | 30/07/2026 | AWS Documentation |

## Kết quả đạt được tuần 7

Sau tuần thứ bảy, hệ thống đã có thể triển khai trên môi trường AWS và tự động cập nhật thông qua GitHub.

Một số kết quả đạt được gồm:

- Triển khai thành công project lên Amazon EC2.
- Cài đặt Docker và Docker Compose trên máy chủ.
- Chạy frontend và backend bằng Docker Compose.
- Thiết lập GitHub Actions để tự động triển khai ứng dụng.
- Cấu hình kết nối với Amazon S3, DynamoDB và SageMaker.
- Khắc phục các lỗi liên quan đến SSH, Docker và môi trường triển khai.
- Hoàn thiện quy trình triển khai cơ bản cho project.