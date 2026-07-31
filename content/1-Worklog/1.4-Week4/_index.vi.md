---
title: "Worklog Tuần 4"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

{{% notice tip %}}
Tuần thứ tư tập trung tìm hiểu về Docker và cách triển khai ứng dụng bằng container. Đồng thời thực hành xây dựng môi trường chạy ứng dụng với Docker Compose nhằm chuẩn bị cho việc triển khai project trên AWS trong các tuần tiếp theo.
{{% /notice %}}

## Mục tiêu tuần 4

- Hiểu khái niệm Container và Docker.
- Phân biệt Virtual Machine và Container.
- Làm quen với Docker Image, Docker Container và Dockerfile.
- Thực hành sử dụng Docker Compose để chạy nhiều service.
- Chuẩn bị môi trường triển khai ứng dụng.

## Các công việc triển khai trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu Docker và Container.<br>- So sánh Docker với Virtual Machine.<br>- Cài đặt Docker Desktop. | 11/08/2026 | 11/08/2026 | https://docs.docker.com/get-started/ |
| 3 | - Tìm hiểu Docker Image, Container và Docker Hub.<br>- Thực hành chạy các Image có sẵn bằng Docker. | 12/08/2026 | 12/08/2026 | https://docs.docker.com/ |
| 4 | - Tìm hiểu Dockerfile.<br>- Thực hành build Image từ Dockerfile.<br>- Chạy ứng dụng bằng Docker Container. | 13/08/2026 | 13/08/2026 | https://docs.docker.com/engine/reference/builder/ |
| 5 | - Tìm hiểu Docker Compose.<br>- Viết file `docker-compose.yml` đơn giản.<br>- Thực hành chạy nhiều container cùng lúc. | 14/08/2026 | 14/08/2026 | https://docs.docker.com/compose/ |
| 6 | - Ôn tập Docker.<br>- Thực hành đóng gói một ứng dụng mẫu bằng Docker và Docker Compose.<br>- Kiểm tra môi trường chạy ứng dụng. | 15/08/2026 | 15/08/2026 | Docker Documentation |

## Kết quả đạt được tuần 4

Sau tuần thứ tư, đã nắm được kiến thức cơ bản về Docker và có thể sử dụng container để triển khai ứng dụng.

Một số kết quả đạt được gồm:

- Hiểu được sự khác nhau giữa Virtual Machine và Container.
- Hiểu vai trò của Docker trong quá trình phát triển và triển khai phần mềm.
- Nắm được các thành phần cơ bản:
  - Docker Image.
  - Docker Container.
  - Docker Registry.
  - Docker Hub.

- Cài đặt và sử dụng thành công Docker Desktop.

- Thực hiện được các thao tác cơ bản:
  - Download Image từ Docker Hub.
  - Tạo và chạy Container.
  - Dừng, khởi động và xóa Container.
  - Xem log của Container.

- Viết được Dockerfile đơn giản để đóng gói ứng dụng.

- Thực hành sử dụng Docker Compose để quản lý nhiều service trong cùng một môi trường.

- Hiểu được quy trình đóng gói ứng dụng bằng Docker, tạo tiền đề cho việc triển khai project lên máy chủ và môi trường Cloud trong các tuần tiếp theo.