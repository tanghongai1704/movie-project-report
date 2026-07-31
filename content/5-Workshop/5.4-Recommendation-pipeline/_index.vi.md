---
title: "Pipeline gợi ý"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Pipeline hỗ trợ nhiều chiến lược để phục vụ từng trạng thái người dùng.

## Các chiến lược gợi ý

- **Popularity:** xếp hạng phổ biến có trọng số theo phong cách IMDb cho khách chưa đăng nhập.
- **Content-based:** sử dụng TF-IDF và cosine similarity cho người dùng mới dựa trên thể loại onboarding.
- **Collaborative filtering:** sử dụng implicit ALS để tạo candidate cho người dùng quay lại.
- **Hybrid ranking:** kết hợp weighted Reciprocal Rank Fusion, độ tương đồng gần đây và business-rule filtering.

![Luồng huấn luyện, đánh giá, quản lý artifact và phục vụ mô hình](/images/5-Workshop/5.4-Recommendation-pipeline/recommendation-pipeline-flow.png)

*Pipeline huấn luyện và đánh giá mô hình, kiểm tra promotion gate, lưu artifact trên S3, sau đó phục vụ qua SageMaker Endpoint và FastAPI provider.*

## Input và output

Input của pipeline gồm:

- Movie features.
- Interaction split theo thời gian.
- Thể loại onboarding.
- Các interaction gần đây.
- Danh sách movie ID cần loại trừ.

Recommendation engine trả:

- Danh sách `movie_id` theo thứ tự.
- `score`.
- `reason_code`.
- `reason_context`.

Backend bổ sung metadata từ bảng `Movies` rồi cache `movie_id`, `score` và `reason_code`.

## Vai trò của SageMaker và EC2

SageMaker Processing Job được triển khai để chạy retraining. EC2 có thể chạy Docker application hoặc tùy chọn chạy retraining định kỳ bằng systemd.

Backend đã có client gọi SageMaker real-time endpoint, nhưng source hiện chưa có endpoint packaging và deployment.


## Nội dung

1. [Chuẩn bị môi trường gợi ý](5.4.1-prepare-environment/)
2. [Huấn luyện, đánh giá và chạy SageMaker Processing Job](5.4.2-train-and-deploy-model/)
3. [Tích hợp ứng dụng trên EC2](5.4.3-integrate-ec2-application/)
4. [Kiểm thử đầu cuối](5.4.4-end-to-end-testing/)
5. [Kiểm thử model](5.4.5-model-testing/)
