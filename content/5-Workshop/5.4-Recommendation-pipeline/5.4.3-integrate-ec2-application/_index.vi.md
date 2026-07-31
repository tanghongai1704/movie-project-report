---
title: "Tích hợp ứng dụng trên EC2"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

Repository không tạo EC2. Workflow GitHub Actions giả định một host đã tồn tại, có thể kết nối SSH và đã được chuẩn bị để chạy Docker Compose.

## 1. Chuẩn bị EC2 host

Platform owner cần cung cấp:

- `<EC2_INSTANCE_ID>`.
- IAM instance profile.
- Application directory trên host.
- Inbound rule cho `<APPLICATION_PORT>` hoặc reverse proxy.
- Dung lượng đĩa.
- DNS và TLS nếu ứng dụng public.
- Quyền truy cập Git.
- Docker Engine và Docker Compose v2.

Các thông tin AMI, instance type, VPC, subnet, security group, disk, DNS và TLS hiện chưa có trong repository.

Ảnh chụp môi trường triển khai xác nhận instance `movie-recommendation-server` đang ở trạng thái `Running`, sử dụng instance type `t3.micro` và có cả public lẫn private IPv4. Đây là bằng chứng của môi trường hiện tại, không thay thế cấu hình Infrastructure as Code.

![Thông tin EC2 instance chạy ứng dụng gợi ý phim](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/ec2-instance-summary.jpg)

*EC2 Console xác nhận trạng thái, instance type, địa chỉ mạng, hostname và VPC của application host.*

![Kết nối SSH thành công tới EC2 host](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/ec2-ssh-session.jpg)

*Phiên SSH xác nhận có thể truy cập Ubuntu 24.04.4 LTS trên EC2. Host đang báo cần restart và còn bản cập nhật, vì vậy cần hoàn tất bảo trì trước khi xem đây là trạng thái production-ready.*

## 2. Cấu hình ứng dụng

Đặt `.env` trực tiếp trên EC2 theo quy trình quản lý secret được phê duyệt. Không commit `.env` và không tạo file này trong GitHub Actions.

Trên EC2, ưu tiên instance profile để AWS SDK nhận credential qua default provider chain. Không sao chép permanent access key lên máy chủ.

![GitHub deploy key dành cho EC2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/github-ec2-deploy-key.jpg)

*Repository đã cấu hình deploy key `EC2 Deploy` ở chế độ read-only để host lấy source code. Private key phải chỉ tồn tại trên host hoặc trong secret store được phê duyệt.*

## 3. Workflow triển khai

Khi push branch `main`, GitHub Actions:

1. Build frontend.
2. Cài dependency backend và chạy `compileall`.
3. SSH vào EC2.
4. Chuyển tới `EC2_APP_DIR`.
5. Pull source từ `main`.
6. Chạy Docker Compose.

<!-- IMAGE-5.4.3-01: GitHub Actions deployment thành công, đã che host/user/secrets. -->

{{% notice warning %}}
Workflow chạy `docker compose pull`, nhưng Compose hiện dùng local build context thay vì image registry. Nếu host không rebuild image, `docker compose up -d` có thể tiếp tục dùng image cũ.
{{% /notice %}}

## 4. Runtime integration

Khi backend khởi động, nó:

1. Tạo boto3 session.
2. Xác minh STS identity.
3. Mô tả key schema của các bảng DynamoDB.
4. Kiểm tra S3 bucket.
5. Mô tả SageMaker endpoint theo cơ chế health check không chặn toàn bộ ứng dụng.
6. Khởi tạo repository, service và provider.

Khi endpoint không khả dụng, guest API vẫn có thể chạy; personalized cache miss trả lỗi có kiểm soát.

## 5. Khởi động ứng dụng

Tại application directory trên EC2, sau khi code, `.env`, Docker và IAM role đã sẵn sàng:

```bash
docker compose config --quiet
docker compose up --build -d
docker compose ps
docker compose logs backend --tail 100
```

## 6. Kiểm tra service

```bash
curl -f "http://127.0.0.1:<BACKEND_PORT>/health"

curl -f \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/movies?limit=1"
```

Kết quả mong đợi:

- Backend container ở trạng thái healthy.
- Frontend trả HTML.
- `/health` trả `{"status":"ok"}`.
- `/movies` trả JSON array, hoặc lỗi `503` có kiểm soát nếu data resource cấu hình sai.
- Startup log không lộ credential.

<!-- IMAGE-5.4.3-02: Docker services và health check trên EC2, đã che IP/hostname. -->

## 7. Phân biệt EC2 application và EC2 retraining

`ml/deploy/ec2_bootstrap.sh` cấu hình một systemd timer cho retraining, không phải web deployment. Template này hiện cần sửa:

- Subdirectory mặc định không trùng path submodule `ml`.
- Event prefix `events/` không trùng cấu hình canonical `datasets/exports/`.

Không sử dụng template cho production trước khi hai điểm này được review.

**Nguồn đối chiếu:** `.github/workflows/deploy.yml`, `docker-compose.yml`, `backend/app/aws/infrastructure.py` và `ml/deploy/ec2_bootstrap.sh`.
