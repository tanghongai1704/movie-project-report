---
title: "Blog 1"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# AMAZON ATHENA - PHÂN TÍCH DỮ LIỆU TRÊN S3 BẰNG SQL MÀ KHÔNG CẦN TẠO DATABASE

Trong quá trình học AWS, mình thường nghĩ rằng muốn truy vấn dữ liệu thì cần phải cài đặt một hệ quản trị cơ sở dữ liệu như MySQL hay PostgreSQL. Tuy nhiên, khi tìm hiểu về **Amazon Athena**, mình nhận ra có một cách đơn giản hơn trong một số trường hợp.

Amazon Athena là dịch vụ serverless cho phép truy vấn dữ liệu lưu trên Amazon S3 bằng cú pháp SQL quen thuộc. Điều này có nghĩa là mình không cần tạo máy chủ, cài đặt database hay quản lý hạ tầng mà vẫn có thể phân tích dữ liệu trực tiếp. Athena sử dụng công cụ truy vấn mã nguồn mở Trino, hỗ trợ nhiều định dạng dữ liệu như CSV, JSON, Parquet, ORC và Avro.

## Amazon Athena có thể dùng để làm gì?

Sau khi tìm hiểu, mình thấy Athena phù hợp với khá nhiều trường hợp như:

- Phân tích file CSV hoặc JSON lưu trên S3.
- Truy vấn log từ CloudTrail, VPC Flow Logs hoặc Application Logs.
- Phân tích dữ liệu phục vụ báo cáo.
- Hỗ trợ xây dựng Data Lake kết hợp với Amazon S3.
- Làm nguồn dữ liệu cho Amazon QuickSight để trực quan hóa dữ liệu.

{{% notice tip %}}
Điểm mình thấy hay là chỉ cần dữ liệu nằm trên S3 là đã có thể sử dụng SQL để truy vấn mà không cần import vào database trước.
{{% /notice %}}

## Truy vấn dữ liệu bằng Amazon Athena

Để hiểu rõ hơn, mình thử làm theo một ví dụ đơn giản với file CSV lưu trên S3.

**Bước 1:** Tạo một S3 Bucket và tải lên file dữ liệu `students.csv`. Nội dung gồm các cột như:
- id
- name
- major
- gpa

**Bước 2:**Truy cập AWS Console và mở Amazon Athena.

Lần đầu sử dụng, Athena sẽ yêu cầu cấu hình một S3 Bucket để lưu kết quả truy vấn.

**Bước 3:** Tạo Database.

```sql
CREATE DATABASE university;
```

**Bước 4:** Tạo bảng tham chiếu đến file trên S3.

```sql
CREATE EXTERNAL TABLE students (
    id INT,
    name STRING,
    major STRING,
    gpa DOUBLE
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION 's3://your-bucket/students/';
```

{{% notice note %}}
Athena chỉ tạo metadata, dữ liệu vẫn nằm trên S3 và không bị sao chép sang nơi khác.
{{% /notice %}}

**Bước 5:** Thực hiện truy vấn.

```sql
SELECT name, gpa
FROM students
WHERE gpa >= 3.5;
```

{{% notice note %}}
Sau vài giây, kết quả sẽ hiển thị ngay trên giao diện Athena.
{{% /notice %}}

## Ưu điểm

Sau khi tìm hiểu, mình thấy Amazon Athena có khá nhiều ưu điểm.

Đầu tiên là không cần cài đặt hay quản lý máy chủ cơ sở dữ liệu. Điều này giúp việc bắt đầu phân tích dữ liệu trở nên nhanh hơn.

Thứ hai là Athena hỗ trợ nhiều định dạng dữ liệu khác nhau. Nếu dữ liệu được lưu dưới dạng Parquet hoặc ORC thì hiệu năng truy vấn cũng sẽ tốt hơn.

Ngoài ra, Athena tích hợp tốt với nhiều dịch vụ AWS như AWS Glue Data Catalog, Amazon QuickSight và Amazon S3, phù hợp để xây dựng các hệ thống phân tích dữ liệu.

## Một số điểm cần lưu ý

Bên cạnh những ưu điểm trên, mình cũng thấy có một số điều cần cân nhắc. Athena được tính phí dựa trên dung lượng dữ liệu được quét trong mỗi truy vấn. Vì vậy, nếu dữ liệu chưa được tối ưu hoặc sử dụng `SELECT *` trên các file rất lớn thì chi phí có thể tăng lên.

Theo tài liệu của AWS, để giảm chi phí và tăng hiệu năng, nên:
- Sử dụng định dạng cột như Parquet hoặc ORC.
- Phân vùng dữ liệu - Partitioning.
- Chỉ truy vấn các cột cần thiết thay vì lấy toàn bộ dữ liệu.

## Khi nào nên sử dụng?

Theo mình, Amazon Athena sẽ phù hợp khi:

- Muốn phân tích dữ liệu lưu trên Amazon S3.
- Cần kiểm tra nhanh các file CSV hoặc JSON.
- Phân tích log của hệ thống.
- Xây dựng Data Lake mà không muốn vận hành database riêng.
- Kết hợp với QuickSight để tạo dashboard.

## Kết luận

Sau khi tìm hiểu, mình thấy Amazon Athena là một service khá tiện lợi cho các bài toán phân tích dữ liệu. Chỉ với Amazon S3 và một vài câu lệnh SQL, mình đã có thể truy vấn dữ liệu mà không cần triển khai thêm một hệ quản trị cơ sở dữ liệu. Mình nghĩ đây là một dịch vụ đáng thử nếu đang học về Data Analytics hoặc làm các dự án cần xử lý dữ liệu trên AWS.

## Tài liệu tham khảo
1. [AWS Documentation – Amazon Athena](https://docs.aws.amazon.com/athena/latest/ug/what-is.html)
2. [Getting Started with Amazon Athena](https://docs.aws.amazon.com/.../ug/getting-started.html)
3. [Amazon Athena User Guide](https://docs.aws.amazon.com/athena/latest/ug/)
4. [Amazon Athena Pricing](https://aws.amazon.com/athena/pricing/)