---
title: "Worklog Tuần 10"
date: 2026-07-27
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

{{% notice tip %}}
Tuần thứ mười tập trung triển khai ứng dụng lên môi trường AWS, cấu hình Docker và kiểm thử hệ thống sau khi triển khai. Đồng thời thực hiện tối ưu một số thành phần nhằm đảm bảo ứng dụng hoạt động ổn định.
{{% /notice %}}

## Mục tiêu tuần 10

- Triển khai ứng dụng lên môi trường AWS.
- Cấu hình Docker để chạy hệ thống.
- Kiểm thử ứng dụng sau khi triển khai.
- Khắc phục các lỗi phát sinh trong quá trình deploy.
- Hoàn thiện tài liệu kỹ thuật của project.

## Các công việc triển khai trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Chuẩn bị môi trường triển khai trên AWS.<br>- Cấu hình EC2 để chạy ứng dụng.<br>- Kiểm tra kết nối đến máy chủ. | 22/09/2026 | 22/09/2026 | AWS EC2 Documentation |
| 3 | - Cài đặt Docker và Docker Compose trên EC2.<br>- Chạy frontend và backend bằng Docker Compose.<br>- Kiểm tra log của các container. | 23/09/2026 | 23/09/2026 | Docker Documentation |
| 4 | - Cấu hình biến môi trường cho ứng dụng.<br>- Kết nối ứng dụng với Amazon S3, DynamoDB và SageMaker.<br>- Kiểm tra hoạt động của các API sau khi triển khai. | 24/09/2026 | 24/09/2026 | AWS Documentation |
| 5 | - Kiểm thử toàn bộ hệ thống trên môi trường triển khai.<br>- Khắc phục các lỗi phát sinh liên quan đến cấu hình và kết nối.<br>- Tối ưu hiệu năng của ứng dụng. | 25/09/2026 | 25/09/2026 | Tài liệu nhóm |
| 6 | - Cập nhật README và tài liệu hướng dẫn triển khai.<br>- Ghi nhận các vấn đề gặp phải và cách xử lý.<br>- Chuẩn bị cho giai đoạn hoàn thiện project. | 26/09/2026 | 26/09/2026 | GitHub Documentation |

## Kết quả đạt được tuần 10

Sau tuần thứ mười, hệ thống đã được triển khai thành công lên môi trường AWS và có thể hoạt động ổn định.

Một số kết quả đạt được gồm:

- Hoàn thành việc triển khai ứng dụng lên Amazon EC2.

- Cài đặt và cấu hình thành công:
  - Docker.
  - Docker Compose.
  - Các biến môi trường cần thiết cho ứng dụng.

- Triển khai frontend và backend bằng Docker Compose.

- Kết nối thành công ứng dụng với các dịch vụ AWS:
  - Amazon S3.
  - Amazon DynamoDB.
  - Amazon SageMaker.

- Kiểm tra các API sau khi triển khai và xác nhận hệ thống hoạt động đúng như mong đợi.

- Khắc phục một số lỗi phát sinh trong quá trình triển khai như lỗi cấu hình môi trường, quyền truy cập IAM và kết nối giữa các dịch vụ.

- Hoàn thiện tài liệu hướng dẫn cài đặt và triển khai, giúp các thành viên khác có thể thiết lập và chạy project dễ dàng hơn.