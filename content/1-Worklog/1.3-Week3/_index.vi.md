---
title: "Worklog Tuần 3"
date: 2026-06-15
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

{{% notice tip %}}
Tuần thứ ba tập trung tìm hiểu về hạ tầng mạng trên AWS, bao gồm Amazon VPC, Subnet, Internet Gateway và Security Group. Đồng thời thực hành xây dựng một môi trường mạng cơ bản để hiểu cách các tài nguyên trên AWS có thể kết nối với nhau.
{{% /notice %}}

## Mục tiêu tuần 3

- Hiểu kiến trúc mạng cơ bản trên AWS.
- Tìm hiểu các thành phần của Amazon VPC.
- Phân biệt Public Subnet và Private Subnet.
- Thực hành tạo và cấu hình một VPC đơn giản.
- Hiểu vai trò của Security Group trong việc kiểm soát truy cập.

## Các công việc triển khai trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu Amazon VPC.<br>- Tìm hiểu CIDR Block và IP Address.<br>- Đọc kiến trúc mạng cơ bản trên AWS. | 04/08/2026 | 04/08/2026 | https://docs.aws.amazon.com/vpc/ |
| 3 | - Tìm hiểu Subnet, Route Table và Internet Gateway.<br>- Phân biệt Public Subnet và Private Subnet. | 05/08/2026 | 05/08/2026 | https://docs.aws.amazon.com/vpc/latest/userguide/ |
| 4 | - Thực hành tạo VPC.<br>- Tạo Public Subnet.<br>- Gắn Internet Gateway và cấu hình Route Table. | 06/08/2026 | 06/08/2026 | AWS Management Console |
| 5 | - Tìm hiểu Security Group và Network ACL.<br>- Cấu hình Security Group cho EC2 Instance.<br>- Kiểm tra kết nối SSH. | 07/08/2026 | 07/08/2026 | https://docs.aws.amazon.com/ec2/ |
| 6 | - Ôn tập toàn bộ kiến thức về Networking.<br>- Thực hành tạo EC2 trong VPC vừa cấu hình và kiểm tra khả năng truy cập từ Internet. | 08/08/2026 | 08/08/2026 | AWS Documentation |

## Kết quả đạt được tuần 3

Sau tuần thứ ba, đã nắm được các kiến thức cơ bản về hạ tầng mạng trên AWS và hiểu cách các tài nguyên được kết nối với nhau.

Một số kết quả đạt được gồm:

- Hiểu vai trò của Amazon VPC trong việc xây dựng mạng riêng trên AWS.
- Hiểu ý nghĩa của CIDR Block và cách phân chia địa chỉ IP.
- Phân biệt được:
  - Public Subnet.
  - Private Subnet.
  - Internet Gateway.
  - Route Table.

- Thực hành tạo thành công:
  - Một Amazon VPC.
  - Public Subnet.
  - Internet Gateway.
  - Route Table.

- Hiểu cách Security Group kiểm soát lưu lượng truy cập đến EC2 Instance.

- Thực hiện cấu hình Security Group để:
  - Cho phép kết nối SSH.
  - Cho phép truy cập HTTP (nếu cần).
  - Kiểm tra khả năng kết nối từ máy cá nhân đến EC2.

- Hiểu được mối quan hệ giữa VPC, Subnet, Route Table và Internet Gateway trong việc xây dựng một hệ thống mạng cơ bản trên AWS.

- Chuẩn bị kiến thức về Networking để phục vụ cho việc triển khai các dịch vụ và ứng dụng ở các tuần tiếp theo.
