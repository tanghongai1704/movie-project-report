---
title: "Tích hợp ứng dụng trên EC2"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

Repository không tạo EC2. Workflow GitHub Actions giả định một host đã tồn tại, có thể kết nối SSH và đã được chuẩn bị để chạy Docker Compose.

## 1. Tạo EC2 instance bằng AWS Console

1. Mở **Amazon EC2** → **Instances** → **Launch instances**.
2. Nhập name `movie-recommendation-server` và thêm tag `Environment=<ENVIRONMENT>`.
3. Chọn AMI **Ubuntu Server 24.04 LTS, 64-bit (x86)** hoặc AMI Linux đã được tổ chức phê duyệt.
4. Chọn instance type phù hợp. Môi trường workshop hiện dùng `t3.micro`; cần tăng cấu hình nếu Docker build hoặc tải thực tế vượt tài nguyên này.
5. Tại **Key pair (login)**, chọn key pair hiện có hoặc tạo key pair mới. Lưu private key một lần tại vị trí an toàn; AWS không cho tải lại private key.
6. Tại **Network settings**, chọn đúng VPC và subnet. Chỉ bật auto-assign public IPv4 khi workshop cần truy cập trực tiếp từ Internet.
7. Chọn hoặc tạo security group dành riêng cho application host; cấu hình inbound rules theo mục 2 bên dưới.
8. Tại **Configure storage**, chọn volume `gp3` có dung lượng đủ cho OS, source code, Docker image, container và log. Bật EBS encryption.
9. Mở **Advanced details** và gắn IAM instance profile của backend; không đưa access key vào user data.
10. Review **Summary** → **Launch instance** → chờ instance state `Running` và cả hai status check đều pass.

<!-- IMAGE-5.4.3-EC2-01: Launch instance summary với AMI, instance type, VPC/subnet, storage và IAM instance profile. -->

## 2. Cấu hình Security Group và inbound rules

1. Mở **EC2** → **Security Groups** → chọn security group gắn với instance.
2. Chọn tab **Inbound rules** → **Edit inbound rules** → **Add rule**.
3. Chỉ thêm các rule thực sự cần thiết theo bảng dưới đây.
4. Chọn **Save rules** và kiểm tra lại từ một client được phép.

| Mục đích | Type/Protocol | Port | Source khuyến nghị |
|---|---|---:|---|
| Quản trị Linux | SSH / TCP | 22 | `<ADMIN_PUBLIC_IP>/32`, VPN CIDR hoặc bastion security group; không dùng `0.0.0.0/0` |
| Web không TLS | HTTP / TCP | 80 | `0.0.0.0/0` và `::/0` chỉ khi website cần public |
| Web có TLS | HTTPS / TCP | 443 | `0.0.0.0/0` và `::/0` khi website cần public |
| Application port trực tiếp | Custom TCP | `<APPLICATION_PORT>` | Security group của load balancer/reverse proxy hoặc CIDR kiểm thử được phê duyệt |
| Backend nội bộ | Custom TCP | `<BACKEND_PORT>` | Không tạo public rule nếu frontend/reverse proxy chạy cùng host |

{{% notice warning %}}
Security group là stateful. Không mở toàn bộ port hoặc mở SSH từ mọi địa chỉ chỉ để xử lý lỗi kết nối. Nếu đã có rule rộng hơn, rule rộng hơn sẽ thắng về phạm vi truy cập.
{{% /notice %}}

<!-- IMAGE-5.4.3-EC2-02: Inbound rules thể hiện SSH giới hạn IP và HTTP/HTTPS hoặc application port đúng phạm vi. -->

## 3. Chuẩn bị EC2 host

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

## 4. Cấu hình ứng dụng

Đặt `.env` trực tiếp trên EC2 theo quy trình quản lý secret được phê duyệt. Không commit `.env` và không tạo file này trong GitHub Actions.

Trên EC2, ưu tiên instance profile để AWS SDK nhận credential qua default provider chain. Không sao chép permanent access key lên máy chủ.

![GitHub deploy key dành cho EC2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/github-ec2-deploy-key.jpg)

*Repository đã cấu hình deploy key `EC2 Deploy` ở chế độ read-only để host lấy source code. Private key phải chỉ tồn tại trên host hoặc trong secret store được phê duyệt.*

## 5. Workflow triển khai

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

## 6. Runtime integration

Khi backend khởi động, nó:

1. Tạo boto3 session.
2. Xác minh STS identity.
3. Mô tả key schema của các bảng DynamoDB.
4. Kiểm tra S3 bucket.
5. Mô tả SageMaker endpoint theo cơ chế health check không chặn toàn bộ ứng dụng.
6. Khởi tạo repository, service và provider.

Khi endpoint không khả dụng, guest API vẫn có thể chạy; personalized cache miss trả lỗi có kiểm soát.

## 7. Khởi động ứng dụng

Tại application directory trên EC2, sau khi code, `.env`, Docker và IAM role đã sẵn sàng:

```bash
docker compose config --quiet
docker compose up --build -d
docker compose ps
docker compose logs backend --tail 100
```

## 8. Kiểm tra service

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

## 9. Phân biệt EC2 application và EC2 retraining

`ml/deploy/ec2_bootstrap.sh` cấu hình một systemd timer cho retraining, không phải web deployment. Template này hiện cần sửa:

- Subdirectory mặc định không trùng path submodule `ml`.
- Event prefix `events/` không trùng cấu hình canonical `datasets/exports/`.

Không sử dụng template cho production trước khi hai điểm này được review.

**Tài liệu AWS chính thức:** [Launch an EC2 instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html), [tạo security group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-security-group.html) và [cấu hình inbound/outbound rules](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/changing-security-group.html).

**Nguồn đối chiếu:** `.github/workflows/deploy.yml`, `docker-compose.yml`, `backend/app/aws/infrastructure.py` và `ml/deploy/ec2_bootstrap.sh`.
