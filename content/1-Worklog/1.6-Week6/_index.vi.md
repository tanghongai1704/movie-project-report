---
title: "Worklog Tuần 6"
date: 2026-07-06
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

{{% notice tip %}}
Tuần thứ sáu tập trung tìm hiểu các dịch vụ hỗ trợ xây dựng ứng dụng serverless trên AWS, bao gồm AWS Lambda và Amazon API Gateway. Đồng thời thực hành xây dựng một API đơn giản để hiểu cách các dịch vụ này hoạt động cùng nhau.
{{% /notice %}}

## Mục tiêu tuần 6

- Tìm hiểu mô hình Serverless trên AWS.
- Hiểu cách hoạt động của AWS Lambda.
- Tìm hiểu Amazon API Gateway và cách xây dựng REST API.
- Thực hành tạo Lambda Function và kết nối với API Gateway.
- Chuẩn bị kiến thức để tích hợp các dịch vụ vào project.

## Các công việc triển khai trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu mô hình Serverless.<br>- Tìm hiểu AWS Lambda và các trường hợp sử dụng phổ biến.<br>- Tạo Lambda Function đầu tiên. | 25/08/2026 | 25/08/2026 | https://docs.aws.amazon.com/lambda/ |
| 3 | - Thực hành viết Lambda Function bằng Python.<br>- Cấu hình Trigger và kiểm tra kết quả thực thi.<br>- Theo dõi log bằng Amazon CloudWatch. | 26/08/2026 | 26/08/2026 | https://docs.aws.amazon.com/lambda/latest/dg/ |
| 4 | - Tìm hiểu Amazon API Gateway.<br>- Tìm hiểu REST API và HTTP API.<br>- Tạo API đơn giản kết nối với Lambda. | 27/08/2026 | 27/08/2026 | https://docs.aws.amazon.com/apigateway/ |
| 5 | - Thực hành gọi API bằng Postman hoặc trình duyệt.<br>- Kiểm tra phản hồi từ Lambda.<br>- Điều chỉnh cấu hình khi cần thiết. | 28/08/2026 | 28/08/2026 | AWS Documentation |
| 6 | - Ôn tập Lambda và API Gateway.<br>- Tìm hiểu khả năng tích hợp với các dịch vụ AWS khác như S3 và DynamoDB.<br>- Chuẩn bị kiến thức để áp dụng vào project. | 29/08/2026 | 29/08/2026 | AWS Documentation |

## Kết quả đạt được tuần 6

Sau tuần thứ sáu, đã nắm được quy trình xây dựng một ứng dụng serverless cơ bản trên AWS.

Một số kết quả đạt được gồm:

- Hiểu được khái niệm Serverless và lợi ích của mô hình này trong phát triển ứng dụng.
- Hiểu vai trò của AWS Lambda trong việc xử lý các tác vụ theo sự kiện.
- Tạo và triển khai thành công Lambda Function bằng Python.
- Thực hiện kiểm thử Lambda Function và theo dõi kết quả thông qua Amazon CloudWatch Logs.

- Hiểu được chức năng của Amazon API Gateway trong việc cung cấp API cho ứng dụng.

- Thực hành:
  - Tạo REST API.
  - Kết nối API Gateway với AWS Lambda.
  - Gửi request và nhận phản hồi thông qua API.

- Làm quen với quy trình xây dựng một API không cần quản lý máy chủ.

- Hiểu khả năng tích hợp giữa Lambda, API Gateway và các dịch vụ AWS khác, tạo nền tảng để phát triển các chức năng của project trong các tuần tiếp theo.