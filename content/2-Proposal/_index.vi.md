---
title: "Bản đề xuất"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Movie Recommendation System
## Hệ thống đề xuất phim dựa trên dữ liệu người dùng sử dụng

### 1. Tóm tắt điều hành
Movie Recommendation System là một hệ thống gợi ý phim dựa trên dữ liệu người dùng, nhằm cung cấp trải nghiệm cá nhân hóa cho người dùng. Hệ thống sử dụng kết hợp Content-Base Filtering và Collaborative Filtering.

Dự án được triển khai toàn diện trên AWS với kiến trúc Real-time Inference. Quá trình huấn luyện mô hình được xử lý ngầm định kỳ qua SageMaker Processing Jobs. Trong khi đó, việc đưa ra kết quả gợi ý tức thì được đảm nhiệm bởi SageMaker Endpoints hoạt động 24/7. Toàn bộ hệ thống được kết nối qua Backend FastAPI cùng cơ sở dữ liệu Amazon S3 và DynamoDB, đảm bảo khả năng phản hồi mượt mà và xuyên suốt cho người dùng.

### 2. Tuyên bố vấn đề
_Vấn đề hiện tại_

Hiện tại, các dịch vụ xem phim trực tuyến phát triển bùng nổ, số lượng phim và chương trình truyền hình sẵn có đã lên tới hàng chục, hàng trăm nghìn tác phẩm.

Vấn đề cốt lõi: 
- Khi có quá nhiều sự lựa chọn, người dùng thường bị quá tải thông tin và mất quá nhiều thời gian (trung bình 15–20 phút) chỉ để duyệt qua danh sách phim mà không chọn được phim phù hợp.

- Việc tìm kiếm thủ công phim và chương trình dựa theo từ khóa hay thể loại không phản ánh đúng sở thích đa dạng và thay đổi theo thời gian của người dùng dần đến trải nghiệm người dùng bị kém đi.

- Người dùng dễ chán nản và rời khỏi ứng dụng nếu không tìm thấy nội dung hấp dẫn trong vài phút đầu tiên.

Do đó, các dịch vụ xem phim cần một hệ thống gợi ý thông minh có khả năng thấu hiểu hành vi và sở thích của người dùng, từ đó cung cấp các đề xuất cá nhân hóa, giúp người dùng tiết kiệm thời gian và nâng cao trải nghiệm xem phim.

_Giải pháp_

Để giải quyết vấn đề trên, nền tảng cần chuyển dịch từ mô hình **cung cấp nội dung thụ động** sang **gợi ý nội dung chủ động**. Hệ thống đề xuất sử dụng các thuật toán Machine Learning để khai phá dữ liệu hành vi ngầm, từ đó mô hình hóa tính cách người xem và phát hiện các mẫu tương đồng giữa hàng ngàn bộ phim. Nhằm đảm bảo hệ thống vận hành tốt và có khả năng mở rộng khi lượng dữ liệu tương tác tăng vọt, giải pháp bắt buộc phải khai thác dịch vụ điện toán đám mây. Bằng việc kết hợp các dịch vụ hạ tầng AWS, dự án thiết lập một luồng dữ liệu hoàn chỉnh: từ thu thập tương tác, lên lịch huấn luyện mô hình định kỳ, đến phân phối kết quả cá nhân hóa với độ trễ cực thấp. 

### 3. Kiến trúc giải pháp

![Kiến trúc tổng thể](/images/2-Proposal/diagram.png)

_Dịch vụ AWS sử dụng_

- Amazon VPC & Internet Gateway: Cung cấp mạng ảo dùng riêng và cổng giao tiếp an toàn, định tuyến luồng yêu cầu từ người dùng vào hệ thống.
- Amazon EC2: Chạy ứng dụng web (Vite + FastAPI) và phục vụ API trực tiếp.
- Amazon DynamoDB: Lưu trữ metadata phim và lịch sử tương tác người dùng theo thời gian thực.
- Amazon S3: Lưu trữ dữ liệu thô và kết quả mô hình đã huấn luyện.
- Amazon SageMaker: Trung tâm xử lý Machine Learning của hệ thống, bao gồm hai thành phần:
  - **SageMaker Processing Job:** Chạy định kỳ để huấn luyện mô hình gợi ý dựa trên dữ liệu tương tác.
  - **SageMaker Endpoint:** Máy chủ dự đoán duy trì 24/7, luôn tải sẵn phiên bản mô hình mới nhất để phục vụ tính toán theo thời gian thực.
- AWS IAM & CloudWatch: Quản lý quyền truy cập phân luồng và giám sát log hệ thống.

_Thiết kế thành phần_
- Tầng Giao diện: Chịu trách nhiệm hiển thị giao diện người dùng, danh mục phim và trực tiếp thu thập các sự kiện tương tác. (click, rate, watch).
- Tầng Ứng dụng: Điều phối trung tâm: tiếp nhận request từ Frontend, truy xuất dữ liệu từ Database và gọi API sang hệ thống Machine Learning.
- Tầng Dữ liệu: Phân tách thành hai luồng lưu trữ chuyên biệt:
    - Dữ liệu nóng: Quản lý bởi DynamoDB, xử lý các truy vấn cần tốc độ cao như cập nhật lịch sử người dùng và thông tin chi tiết phim.
    - Dữ liệu lưu trữ: Quản lý bởi S3, dùng làm kho chứa an toàn cho các tập dữ liệu thô khổng lồ và lưu trữ các phiên bản tệp trọng số. (Model Artifacts).
- Tầng Học máy: Hoạt động độc lập trên Amazon SageMaker với hai tác vụ riêng biệt là huấn luyện lại mô hình và phục vụ dự đoán thời gian thực.

### 4. Triển khai kỹ thuật
_Các giai đoạn triển khai_

Dự án gồm 2 phần được triển khai song song: xây dựng Web xem phim và xây dựng mô hình Machine Learning gợi ý phim.

1. **Khởi tạo hạ tầng và Chuẩn bị dữ liệu:** Thiết lập nền tảng cơ bản cho cả hai phần Web và Machine Learning, chốt schema dữ liệu và xử lý xong nguồn dữ liệu thô.
    - _Phần Web:_
        - Thiết lập cấu trúc dự án, cấu hình môi trường Docker và các CI/CD cơ bản.  
        - Dựng khung Backend FastAPI, thiết kế schema cho DynamoDB và tạo các bảng trên AWS.  
        - Thiết kế UI/UX cơ bản, dựng luồng Đăng ký/Đăng nhập và Onboarding trên Vite. 
    - _Phần Machine Learning:_
        - Tiền xử lý tập The Movies Dataset.
        - Phát triển Data Pipeline xuất dữ liệu làm sạch ra các tập Train/Validation/Test.

2. **Tính toán chi phí:** Sử dụng AWS Pricing Calculator để ước tính và điều chỉnh

3. **Phát triển tính năng:** Hoàn thiện các tính năng, xây dựng kịch bản người dùng trên Web và huấn luyện thành công các mô hình gợi ý đầu tiên.
    - _Phần Web:_
        - Phát triển trang chi tiết phim, xây dựng API hiển thị metadata.
        - Xây dựng hoàn chỉnh Interaction Pipeline: bắt các sự kiện (như `click`, `watch`, `rate`, `like`) từ Frontend và lưu vào bảng Interactions trên DynamoDB.
    - _Phần Machine Learning:_
        - Xây dựng mô hình **Popularity Ranker** cho người dùng khách và **Content-Based Recommender** cho người dùng đã đăng nhập.
        - Phát triển mô hình cốt lõi **Collaborative Filtering**, chuyển đổi các sự kiện tương tác thành điểm trọng số.
        - Tích hợp mô hình vào SageMaker Endpoint để phục vụ dự đoán gợi ý phim.

4. **Tích hợp hệ thống và Triển khai Cloud:** Xây dựng API POST trên Backend để định tuyến yêu cầu. Đóng gói mô hình AI và triển khai lên SageMaker Endpoint để phục vụ dự đoán thời gian thực. Thiết lập tự động hóa quy trình re-train mô hình định kỳ.
    - _Phần Web:_
        - Tích hợp mô hình Machine Learning vào tiến trình Backend. Xây dựng API POST để định tuyến yêu cầu từ Frontend xuống mô hình.
    - _Phần Machine Learning:_
        - Tự động hóa quy trình re-train mô hình.
        - Chạy thử nghiệm trên Amazon SageMaker.

5. **Kiểm thử:** Rà soát toàn bộ hệ thống, xử lý lỗi và đo lường hiệu năng thực tế. Tối ưu hóa thời gian tải trang và tốc độ truy vấn DynamoDB. Giám sát chi phí AWS để đảm bảo không có tài nguyên chạy ngầm vượt ngân sách. 

_Yêu cầu kỹ thuật_

- _Frontend:_ Sử dụng Vite và có hiểu biết về EC2. Xây dựng giao diện hiển thị phim, quản lý trạng thái đăng nhập.
- _Backend:_ Xây dựng bằng FastAPI. Xử lý chuẩn xác luồng xác thực, quản lý luồng thu thập tương tác, và định tuyến kịch bản gợi ý dựa trên trạng thái người dùng.
- _Machine Learning:_ Phát triển bằng Python sử dụng các thư viện implicit, scikit-learn, numpy, pandas. Yêu cầu xây dựng mô hình Implicit ALS, cùng thuật toán lai ghép Weighted RRF.
- _Cloud - AWS:_ Amazon S3 để lưu trữ tập dữ liệu thô và các file kết quả mô hình. Amazon DynamoDB làm cơ sở dữ liệu chính cho Web. Sử dụng Amazon SageMaker để chạy tự động quy trình Re-train.

### 5. Lộ trình & Mốc triển khai
- _Trước thực tập (Tháng 0):_ Lập kế hoạch, chuẩn bị dataset.
- _Thực tập (Tháng 1-3):_
    - Tháng 1: Tìm hiểu chung các dịch vụ AWS.
    - Tháng 2: Xây dựng web xem phim và hệ thống gợi ý phim.
    - Tháng 3: Kiểm thử hệ thống và chuẩn bị cho triển khai thực tế.
- _Sau triển khai:_ Theo dõi hiệu suất, tối ưu hóa mô hình và mở rộng tính năng.

### 6. Ước tính ngân sách
Có thể xem chi phí trên [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=26e628729fcf910d969fbf894cec6b86db18ad4c)

_Chi phí hạ tầng_

- Amazon EC2: 3,87 USD/tháng (1 t3.micro, 730 giờ, 30 GB storage).
- Amazon SageMaker: 
    - Processing Jobs: 1,41 USD/tháng (1 ml.m5.xlarge, 30 jobs, 10 phút/job).
    - Endpoints: 85,56 USD/tháng (1 ml.m5.xlarge, 730 giờ).
- Amazon DynamoDB: 0,37 USD/tháng (On-demand, 1 GB storage, 100.000 requests).
- Amazon S3 Standard: 0,13 USD/tháng (5 GB storage, 2000 requests).

_Tổng:_ 91.34 USD/tháng, 1,096.08 USD/12 tháng.

### 7. Đánh giá rủi ro

_Ma trận rủi ro_

- **Bùng nổ chi phí Cloud - Ảnh hưởng cao, Xác suất trung bình:** Cấu hình máy chủ dự đoán ml.m5.large chạy 24/7 vượt quá lưu lượng cần thiết, hoặc không cấu hình vòng đời cho Amazon S3 khiến các tệp trọng số cũ tích tụ vĩnh viễn gây tốn phí lưu trữ ngầm.
- **Suy giảm chất lượng mô hình - Ảnh hưởng cao, Xác suất khá:** Quy trình Processing Jobs định kỳ chạy trên tập dữ liệu tương tác bị nhiễu, sinh ra mô hình kém chất lượng và tự động ghi đè lên Endpoint đang hoạt động tốt.
- **Sai lệch dữ liệu nội bộ - Ảnh hưởng cao, Xác suất thấp:** Các tệp ánh xạ chỉ mục không khớp với ma trận mô hình sẽ khiến hệ thống trả về sai ID phim, gây lỗi hiển thị trên giao diện người dùng.

_Chiến lược giảm thiểu_

- **Hàng rào ngân sách và Tối ưu tài nguyên:** Cài đặt AWS Budgets để gửi cảnh báo tự động khi chi phí đạt mức 50% và 75% ngân sách tháng. Áp dụng chính sách S3 Lifecycle Rule để tự động xóa các phiên bản mô hình cũ sau 30 ngày.
- **Cổng kiểm duyệt tự động:** Mô hình mới chỉ được phép đưa vào sử dụng khi thỏa mãn 3 điều kiện:
    - Số lượng người dùng được chấm điểm trên 1000.
    - Chỉ số hiệu suất vượt qua mức cơ sở Popularity Baseline.
    - Không bị sụt giảm quá 5% độ chính xác so với phiên bản hiện tại.
- **Xác thực dữ liệu chéo:** Bổ sung các bước kiểm tra trên Backend để đảm bảo mọi ID phim mà mô hình đưa ra đều tồn tại thực sự trong danh mục DynamoDB.

_Kế hoạch dự phòng_

- **Cơ chế Fallback an toàn:** Nếu Endpoint bị quá tải hoặc luồng người dùng mới chưa đủ dữ liệu tính toán, Backend sẽ tự động lùi về kịch bản an toàn: Gợi ý các bộ phim phổ biến Top-Rated đọc trực tiếp từ DynamoDB để đảm bảo trải nghiệm liền mạch.
- **Quay lui mô hình - Rollback:** Khi phát hiện mô hình mới gợi ý thiếu chính xác, quản trị viên có thể cập nhật cấu hình SageMaker Endpoint trỏ về tệp trọng số cũ. Endpoint sẽ tự động tải lại mô hình ổn định mà không cần khởi động lại toàn bộ hệ thống Backend.

### 8. Kết quả kỳ vọng
_Cải tiến kỹ thuật:_ 

- Triển khai thành công kiến trúc Real-time Inference với SageMaker Endpoint hoạt động 24/7, cung cấp khả năng cá nhân hóa tức thì và loại bỏ hoàn toàn độ trễ khởi động.
- Hệ thống có khả năng phản ứng ngay lập tức với sự thay đổi trong sở thích của người dùng bằng cách bắt các tương tác ngầm (thời lượng xem, lượt chia sẻ, thích/không thích) và truyền trực tiếp vào máy chủ dự đoán.

_Giá trị dài hạn:_ 

- Xây dựng thành công nền tảng kiến trúc vững chắc, đạt tiêu chuẩn vận hành thực tế trên hệ sinh thái AWS.
- Mở ra khả năng tái sử dụng linh hoạt toàn bộ luồng thu thập dữ liệu và suy luận thời gian thực cho các dự án thương mại điện tử, giáo dục trực tuyến hoặc các nền tảng nội dung quy mô lớn khác trong tương lai.