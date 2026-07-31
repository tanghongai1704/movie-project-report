---
title: "Workshop"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

Workshop này trình bày kiến trúc và quy trình vận hành của một hệ thống gợi ý phim được xây dựng bằng **React**, **FastAPI** và các dịch vụ AWS. Hệ thống sử dụng:

![Giao diện chính của hệ thống Streamverse](/images/5-Workshop/ui-home-page.png)

*Giao diện chính của ứng dụng xem phim và gợi ý nội dung.*

- **Amazon DynamoDB** để lưu thông tin phim, tài khoản, hành vi người dùng và bộ nhớ đệm gợi ý.
- **Amazon S3** để lưu dữ liệu thô, dữ liệu đã xử lý, tập huấn luyện, artifact mô hình và báo cáo đánh giá.
- **Amazon SageMaker Processing Job** để chạy quy trình tái huấn luyện theo yêu cầu.
- **Amazon SageMaker Runtime** làm đích gọi suy luận thời gian thực của backend.
- **Amazon EC2** để chạy ứng dụng bằng Docker Compose.
- **AWS IAM** để phân tách quyền của người triển khai, ứng dụng và SageMaker.

## Bài toán được giải quyết

Hệ thống hỗ trợ ba tình huống chính:

1. **Khách chưa đăng nhập** xem danh sách phim phổ biến.
2. **Người dùng mới** chọn thể loại yêu thích trong bước onboarding.
3. **Người dùng quay lại** nhận danh sách gợi ý dựa trên lịch sử tương tác, cache và recommendation provider.

## Phạm vi đã được xác nhận

- DynamoDB có năm bảng logic: `Movies`, `PopularMovies`, `Users`, `UserInteractions` và `RecommendationCache`.
- S3 được chia thành các vùng dữ liệu thô, dữ liệu đã xử lý, tập train, dữ liệu inference, model, báo cáo đánh giá và interaction export.
- Pipeline ML sử dụng **implicit ALS** cho collaborative filtering và có bước đánh giá, promotion gate.
- Backend đã có mã gọi một SageMaker real-time endpoint.
- GitHub Actions triển khai ứng dụng lên một máy chủ EC2 đã tồn tại.
- Mã ứng dụng sử dụng credential provider chain mặc định của AWS SDK.

{{% notice warning %}}
Repository hiện chưa chứa Infrastructure as Code, script nạp dữ liệu vào DynamoDB, SageMaker serving handler, script tạo Model/EndpointConfig/Endpoint, cấu hình IAM hoàn chỉnh, quy trình provision EC2 hoặc cleanup automation. Vì vậy workshop sẽ mô tả trung thực những phần đã có và đánh dấu rõ các bước cần được bổ sung.
{{% /notice %}}

## Kết quả học tập

Sau khi hoàn thành workshop, bạn có thể:

- Giải thích luồng dữ liệu, luồng huấn luyện và luồng suy luận tại thời điểm request.
- Kiểm tra schema của năm bảng DynamoDB và cấu trúc prefix trên S3.
- Chạy data pipeline, validation và dry-run cho quá trình tái huấn luyện.
- Huấn luyện, đánh giá mô hình cục bộ hoặc gửi SageMaker Processing Job khi có đủ quyền.
- Khởi động ứng dụng và kiểm tra các luồng guest, authentication và interaction.
- Chẩn đoán cache hit, lỗi endpoint và lỗi phân quyền.
- Lập kế hoạch dọn tài nguyên theo đúng thứ tự phụ thuộc.

## Nội dung workshop

1. [Kiến trúc và luồng xử lý tổng thể](5.1-Workshop-overview/)
2. [Điều kiện chuẩn bị](5.2-Prerequisites/)
3. [Tầng dữ liệu với S3 và DynamoDB](5.3-Data-layer/)
4. [Pipeline gợi ý](5.4-Recommendation-pipeline/)
5. [IAM và bảo mật](5.5-IAM-security/)
6. [Tổng kết và dọn dẹp tài nguyên](5.6-Cleanup/)

{{% notice note %}}
Không đưa access key, secret key, JWT hoặc nội dung file `.env` vào report và ảnh chụp màn hình.
{{% /notice %}}
