---
title: "Đánh giá và định hướng tương lai"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

Phần này tổng hợp mức độ hoàn thiện của workshop hệ thống gợi ý phim và đề xuất lộ trình phát triển tiếp theo. Đánh giá dựa trên source code, cấu hình, ảnh chụp AWS Console và các kịch bản kiểm thử hiện có; đây chưa phải là chứng nhận SLA cho môi trường production.

## Đánh giá kết quả workshop

| Hạng mục | Mức độ hiện tại | Nhận xét |
|---|---|---|
| Tầng dữ liệu | Đã mô tả và có bằng chứng triển khai | S3 lưu dataset, model artifact và báo cáo; DynamoDB phục vụ catalog, interaction, cache và trạng thái người dùng. |
| Pipeline gợi ý | Hoàn thành ở mức workshop | Có popularity, content-based, implicit ALS, hybrid ranking, offline evaluation và promotion gate. |
| SageMaker | Đã có bằng chứng vận hành | Workshop ghi nhận Processing Job và Endpoint; quy trình đóng gói, phát hành model vẫn cần được tự động hóa đầy đủ. |
| Tích hợp ứng dụng | Đã mô tả, kiểm thử một phần | FastAPI provider hỗ trợ cache, fallback và gọi endpoint; triển khai EC2 cần bổ sung bằng chứng vận hành và kiểm thử tải. |
| IAM và bảo mật | Có baseline | Đã xác định trust policy, least privilege và các quyền cần thiết; vẫn cần policy review tự động và giám sát liên tục. |
| Kiểm thử đầu cuối | Có các luồng chính | Đã bao phủ guest, người dùng mới, người dùng quay lại, interaction và kiểm tra endpoint; chưa thay thế bộ kiểm thử production. |

## Điểm mạnh

- Phân tách rõ batch path trên S3 và request-time path trên DynamoDB.
- Hỗ trợ nhiều chiến lược gợi ý phù hợp với trạng thái người dùng khác nhau.
- Có bước đánh giá offline và promotion gate trước khi phát hành model.
- Có cache, fallback và `reason_code` để tăng khả năng giải thích và khả năng chịu lỗi.
- Workshop chỉ ra ranh giới IAM và thứ tự cleanup nhằm giảm rủi ro bảo mật và chi phí.

## Hạn chế hiện tại

- Chưa có một bộ Infrastructure as Code thống nhất để tái tạo S3, DynamoDB, SageMaker, IAM và EC2.
- Quy trình build và triển khai serving container cho SageMaker Endpoint chưa được thể hiện đầy đủ trong source đã khảo sát.
- Phần lớn bằng chứng triển khai và kiểm thử vẫn được thu thập thủ công.
- Chỉ số offline chưa đủ để kết luận mức độ hài lòng thực tế của người dùng.
- Chưa đánh giá đầy đủ data drift, model drift, fairness, quyền riêng tư và chất lượng dữ liệu dài hạn.
- Chưa có số liệu hoàn chỉnh về tải đồng thời, độ trễ P95/P99, khả năng phục hồi và chi phí trên mỗi request.

## Định hướng phát triển

### Ngắn hạn

1. Chuẩn hóa hạ tầng bằng AWS CDK, CloudFormation hoặc Terraform và tách cấu hình theo môi trường.
2. Tự động hóa data validation, unit test, integration test và end-to-end test trong CI/CD.
3. Bổ sung CloudWatch dashboard, alarm, log retention, cost budget và cảnh báo lỗi pipeline.
4. Hoàn thiện runbook triển khai, rollback, backup, restore và cleanup.
5. Bổ sung liên kết source code, repository và video demo vào mục **8. References**.

### Trung hạn

1. Điều phối train, evaluation và promotion bằng SageMaker Pipelines hoặc một workflow có trạng thái tương đương.
2. Quản lý phiên bản và phê duyệt model bằng model registry; triển khai canary hoặc blue/green.
3. Lập lịch export interaction và retraining, đồng thời theo dõi data drift và model drift.
4. Thực hiện A/B testing để so sánh model mới với baseline trước khi mở rộng traffic.
5. Tối ưu cache, autoscaling và batch inference dựa trên tải thực tế.

### Dài hạn

1. Thử nghiệm mô hình two-tower, sequential recommendation hoặc learning-to-rank khi dữ liệu đủ lớn.
2. Bổ sung feature store hoặc cơ chế quản lý feature nhất quán giữa training và inference.
3. Xây dựng event pipeline gần thời gian thực cho interaction nếu yêu cầu cập nhật gợi ý tăng cao.
4. Hoàn thiện quản trị dữ liệu, bảo vệ thông tin cá nhân, audit trail và quy trình phê duyệt truy cập.
5. Thiết kế disaster recovery và kiểm thử khả năng phục hồi theo mục tiêu RTO/RPO được phê duyệt.

## Chỉ số nên theo dõi

- **Chất lượng model:** Recall@K, NDCG@K, MAP@K, coverage, diversity và novelty.
- **Trải nghiệm người dùng:** click-through rate, tỷ lệ bắt đầu xem, thời lượng xem và tỷ lệ hoàn tất.
- **Vận hành:** P50/P95/P99 latency, error rate, fallback rate, cache hit rate và endpoint availability.
- **Pipeline:** tỷ lệ job thành công, thời gian train, thời gian promotion và độ mới của model.
- **Chi phí và bảo mật:** chi phí theo request hoặc người dùng, tài nguyên nhàn rỗi, phát hiện IAM và số sự kiện truy cập bất thường.

