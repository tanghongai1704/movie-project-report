---
title: "Tầng dữ liệu với S3 và DynamoDB"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

Amazon S3 và Amazon DynamoDB đảm nhận hai vai trò khác nhau:

- **S3** chứa dataset và artifact có kích thước lớn, được sử dụng trong các quy trình batch.
- **DynamoDB** phục vụ lookup và lưu trạng thái trong request path của ứng dụng.

![Luồng xử lý dữ liệu từ Kaggle CSV tới S3 và DynamoDB](/images/5-Workshop/5.3-Data-layer/data-ingestion-flow.jpg)

*Luồng xử lý dữ liệu cục bộ, tạo artifact và đồng bộ lên S3. DynamoDB loader vẫn là thành phần chưa có trong repository.*

## Cấu trúc dữ liệu trên S3

Các vùng logic gồm:

- `raw`: dữ liệu CSV ban đầu.
- `processed`: dữ liệu đã làm sạch và feature.
- `training`: các tập train, validation và test.
- `inference`: lookup artifact cho ML engine.
- `models`: model artifact được quản lý theo version.
- `evaluation`: báo cáo đánh giá và promotion.
- `interaction exports`: feedback từ DynamoDB phục vụ retraining.

Backend HTTP handler không tải dataset từ S3 trong request path.

## Năm bảng DynamoDB

| Bảng | Khóa chính | Mục đích |
|---|---|---|
| `Movies` | `movie_id` | Metadata phim |
| `PopularMovies` | `list_id` | Danh sách phim phổ biến cho guest |
| `Users` | `user_id` | Tài khoản, profile và onboarding |
| `UserInteractions` | `user_id`, `interaction_key` | Click, watch, rating, reaction và share |
| `RecommendationCache` | `user_id`, `scenario` | Cache kết quả gợi ý theo ngữ cảnh |

`Movies` là nguồn metadata chính. `PopularMovies` và `RecommendationCache` chỉ giữ tham chiếu tới phim thay vì nhân bản toàn bộ metadata.

## Quan hệ với hệ thống gợi ý

1. Guest ranking đọc `PopularMovies` rồi bổ sung metadata từ `Movies`.
2. Hành vi của người dùng được ghi vào `UserInteractions`.
3. Interaction có thể được export sang S3 cho lần retrain tiếp theo.
4. Cache cá nhân hóa lưu `movie_id`, `score` và `reason_code`.
5. Kết quả từ cache hoặc provider được enrich bằng `Movies` trước khi trả về API.

## Các bước tiếp theo

1. [Kiểm tra và chuẩn bị tài nguyên lưu trữ](5.3.1-provision-storage/)
2. [Xử lý, tải và xác minh dữ liệu](5.3.2-load-and-validate-data/)

