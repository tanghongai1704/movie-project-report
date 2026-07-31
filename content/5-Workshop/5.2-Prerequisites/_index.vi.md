---
title: "Điều kiện chuẩn bị"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## 1. Công cụ cần thiết

Môi trường thực hành cần:

- Git có hỗ trợ submodule.
- Docker Engine và Docker Compose v2.
- AWS CLI v2.
- Python 3.11.
- Node.js 20 và npm.
- Quyền truy cập Internet để cài dependency.

Backend container sử dụng Python 3.11; frontend Dockerfile sử dụng Node.js 20.

## 2. Khởi tạo repository và ML submodule

ML được quản lý dưới dạng Git submodule. Tại thư mục gốc repository, chạy:

```bash
git submodule update --init --recursive
```

Sau khi hoàn tất, kiểm tra:

```bash
test -f ml/train.py
```

Lệnh phải trả về exit code `0`.

## 3. Xác minh AWS identity

Ưu tiên sử dụng **IAM Identity Center/SSO** cho developer và IAM role cho EC2/SageMaker. Không lưu access key dài hạn trong Git.

```bash
aws sts get-caller-identity --region "<AWS_REGION>"
```
![AWS identity](/images/5-Workshop/5.3-Step-by-step/aws-identity.jpg)

## 4. Tài nguyên phải tồn tại

Repository không tự provision hạ tầng. Trước khi thực hành cần có:

- Một S3 bucket cùng các prefix logic trong `.env.example`.
- Năm bảng DynamoDB với đúng key schema.
- Một `PopularMovies list_id` hợp lệ.
- Một SageMaker endpoint tương thích và ở trạng thái `InService` nếu kiểm tra personalized cache miss.
- Một EC2 đã được provision nếu sử dụng workflow deploy của GitHub.

## 5. Chuẩn bị dataset

Các file CSV được khai báo trong `ml/configs/data_pipeline.yaml`, gồm dữ liệu metadata, ratings, links, credits, keywords và các tập nhỏ dùng cho profiling. Dữ liệu thô không được commit vào Git; hãy đặt chúng đúng thư mục input được cấu hình trong ML project.

## 6. Cấu hình biến môi trường

Tại thư mục gốc repository:

```bash
cp .env.example .env
```

Điền các giá trị theo môi trường:

- `<AWS_REGION>`
- `<JWT_SECRET_VALUE>`
- Tên của năm bảng DynamoDB
- `<POPULAR_LIST_ID>`
- `<S3_BUCKET_NAME>` và các prefix
- `<SAGEMAKER_ENDPOINT_NAME>`
- Frontend API URL và TMDB poster URL

Sau đó kiểm tra Docker Compose:

```bash
docker compose config --quiet
```

Kết quả mong đợi: exit code `0` và `.env` không xuất hiện trong `git status`.


## 7. Cài dependency

```bash
cd backend
python -m pip install -r requirements.txt

cd ../frontend
npm ci

cd ../ml
python -m pip install -r requirements.txt -r requirements-aws.txt
```

## 8. Ma trận quyền tối thiểu

| Principal | Quyền cần thiết |
|---|---|
| Backend runtime | STS identity; DynamoDB describe/read/write/query/scan theo access pattern; S3 list cho startup validation; SageMaker describe/invoke |
| S3/ML tooling | List/Get/Put trên các prefix được phê duyệt |
| SageMaker launcher | Tạo và mô tả Processing Job; `iam:PassRole` cho đúng execution role |
| SageMaker execution role | Đọc/ghi đúng S3 prefix và ghi CloudWatch Logs |
| EC2 application | Sử dụng instance profile thay cho static credential |
