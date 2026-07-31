---
title: "Kiến trúc và luồng xử lý tổng thể"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

Hệ thống gồm giao diện **React/Vite**, backend **FastAPI**, năm bảng DynamoDB, một S3 bucket chứa dữ liệu và artifact, recommendation provider và hai cách chạy tái huấn luyện: cục bộ/EC2 hoặc SageMaker Processing Job.

## Kiến trúc tổng thể

![Kiến trúc tổng thể](/images/5-Workshop/5.1-Workshop-overview/backend-request-flow.jpg)

## Luồng xử lý của ứng dụng

1. Trình duyệt gọi các service tập trung của frontend.
2. `apiClient` gắn base URL, JSON header và JWT đối với endpoint cần xác thực.
3. FastAPI router xác thực request rồi gọi service tương ứng.
4. Service áp dụng business rule và gọi repository hoặc `RecommendationProvider`.
5. Repository thực hiện thao tác DynamoDB; provider gọi SageMaker Runtime.
6. Các `movie_id` được bổ sung metadata từ bảng `Movies` trước khi trả về frontend.

### Luồng dành cho khách

Luồng guest chỉ đọc `PopularMovies`, sau đó dùng `BatchGetItem` để lấy metadata từ `Movies`. Luồng này không cần gọi SageMaker.

### Luồng gợi ý cá nhân hóa

Backend kiểm tra `RecommendationCache` trước. Nếu cache còn hiệu lực, kết quả được trả về mà không gọi endpoint. Khi cache miss, backend dựng request context, gọi SageMaker Runtime, kiểm tra response, lưu cache theo cơ chế best effort và bổ sung metadata phim.

![Luồng request gợi ý qua cache và SageMaker endpoint](/images/5-Workshop/5.1-Workshop-overview/backend-request-flow.jpg)

*Luồng request gợi ý: kiểm tra cache, gọi SageMaker khi cache miss và lấy metadata từ bảng Movies.*

## Luồng huấn luyện

1. Các file Kaggle CSV được profile, làm sạch và ánh xạ MovieLens ID sang TMDB movie ID.
2. Pipeline tạo content features, interactions và các tập dữ liệu chia theo thời gian.
3. Mô hình ALS được huấn luyện và đánh giá offline.
4. Promotion gate quyết định có cập nhật con trỏ `LATEST.json` hay không.
5. Artifact và báo cáo được đồng bộ lên S3.
6. Interaction trong môi trường vận hành có thể được export từ DynamoDB để dùng cho lần retrain tiếp theo.

## Vai trò của từng dịch vụ

| Thành phần | Vai trò |
|---|---|
| Amazon S3 | Lưu trữ bền vững dataset, artifact mô hình và báo cáo |
| Amazon DynamoDB | Lưu metadata, tài khoản, interaction và cache tại request time |
| SageMaker Processing Job | Chạy batch retraining |
| SageMaker Runtime | Đích gọi của recommendation provider trong backend |
| Amazon EC2 | Chạy ứng dụng web bằng Docker (React + FastAPI) và lịch tái huấn luyện |
| AWS IAM | Phân tách quyền deploy, runtime và SageMaker execution theo nguyên tắc tối thiểu |
| Amazon VPC & Internet Gateway | Cung cấp ranh giới mạng (Public Subnet) và định tuyến Internet cho truy cập HTTPS public cùng deployment SSH từ GitHub Actions |
| Amazon CloudWatch | Thu thập log của container ứng dụng và thông số vận hành SageMaker Processing Job |
| AWS Budgets | Theo dõi chi phí cloud so với hạn mức ngân sách ($200 AWS credit guardrail) |

<!-- ## Ranh giới chưa hoàn chỉnh

{{% notice warning %}}
Repository chưa có serving handler hoặc gói triển khai để biến `RecommendationEngine` thành SageMaker endpoint. Có thể kiểm tra riêng local engine và hợp đồng gọi endpoint của backend, nhưng chưa thể dựng mới real-time endpoint chỉ bằng source hiện tại.
{{% /notice %}}

Training path và request path chỉ tạo thành một luồng triển khai khép kín sau khi bổ sung serving handler, model bundle và script tạo SageMaker Model, EndpointConfig và Endpoint.

## Kiểm tra kiến thức

- Guest path không đi qua SageMaker.
- Model chỉ trả movie reference, score và reason; metadata được lấy từ `Movies`.
- SageMaker Processing Job không đồng nghĩa với endpoint deployment.
- Interaction API chỉ ghi hành vi, không trực tiếp chạy recommendation. -->

