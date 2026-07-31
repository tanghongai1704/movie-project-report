---
title: "Kiểm thử mô hình gợi ý"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4.5. </b> "
---

{{% notice info %}}
Web xem phim của nhóm chưa có nhiều dữ liệu lịch sử tương tác của người dùng như `like`, `watch progress`, `share`, ... Do đó, việc kiểm thử mô hình gợi ý dựa trên dữ liệu lịch sử của người dùng sẽ được thực hiện bằng cách tạo ra các dữ liệu giả lập đối với `user_id` trong tập dữ liệu có sẵn. Các `user_id` này đã có lịch sử tương tác `rating` với các bộ phim.
{{% /notice %}}

## Tính năng cần kiểm thử
- Gợi ý 5 phim tương đồng theo bộ phim mà người dùng nhập - **Đối với tất cả loại người dùng**.
- Gợi ý phim **Top-Rated** - **Đối với người dùng chưa đăng nhập**.
- Gợi ý phim dựa trên lịch sử tương tác của người dùng với các bộ phim - **Đối với người dùng đã đăng nhập và có lịch sử tương tác**.
- Gợi ý phim cho người dùng mới đăng nhập bằng cách khảo sát thể loại yêu thích - **Đối với người dùng mới đăng nhập và chưa có lịch sử tương tác**.

![test model 1](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/test-model-1.png)

> Đây là demo kiểm thử hệ thống gợi ý phim

Khi đăng nhập bằng `user_id`, sẽ có 3 loại người dùng:

- **Người dùng chưa đăng nhập - Guest:** Bỏ trống, không nhập `user_id`.  
- **Người dùng mới đăng nhập và chưa có lịch sử tương tác - New user:** Nhập `user_id` từ id 270897 trở lên. Do đã có sẵn dữ liệu lịch sử rating phim của 270896 người dùng trước đó.
- **Người dùng đã đăng nhập và có lịch sử tương tác - Returning user:** Nhập `user_id` từ id 1 đến 270896. Do đã có sẵn dữ liệu lịch sử rating phim của 270896 người dùng trước đó. 

## Guest
### Gợi ý phim tương đồng theo phim mà người dùng nhập

![guest 1](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/guest-1.png)

Người dùng nhập tên phim cần tìm kiếm, hệ thống sẽ hiển thị danh sách kết quả tìm kiếm. Sau đó, người dùng chọn một bộ phim cụ thể trong danh sách để hệ thống gợi ý 5 bộ phim tương đồng.

![guest 2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/guest-2.png)

{{% notice note %}} 
Tuy là người dùng khách nhưng vẫn có thể dùng kịch bản của `returning_user`. Vì hệ thống đã chọn mô hình **Content-based** (do người dùng đang là khách) không cần dữ liệu lịch sử tương tác của người dùng mà chỉ dựa vào nội dung, tên phim mà đã tìm kiếm.
{{% /notice %}}

### Gợi ý phim **Top-Rated**

![guest 3](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/guest-3.png)

Mô hình gợi ý dựa trên **Gợi ý phi cá nhân hóa** để gợi ý phim **Top-Rated** cho người dùng chưa đăng nhập. Hệ thống sử dụng các bảng xếp hạng phim đã được tính toán từ trước.  Cụ thể, danh sách **Top-Rated** được xây dựng dựa trên thuật toán xếp hạng trọng số theo IMDb.

## New user

{{% notice note %}}
Đối với người dùng mới đăng nhập, cũng có tính năng [**Gợi ý phim tương đồng theo phim mà người dùng nhập**](#gợi-ý-phim-tương-đồng-theo-phim-mà-người-dùng-nhập) như người dùng khách.
{{% /notice %}}

### Gợi ý phim theo thể loại yêu thích

![new user 1](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/new-user-1.png)

Người dùng mới đăng nhập sẽ được khảo sát thể loại yêu thích. Sau khi chọn thể loại, hệ thống sẽ gợi ý các bộ phim thuộc thể loại đó.

**Ví dụ:** chọn thể loại `Music`, `Romance`, `Family`

![new user 2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/new-user-2.png)

{{% notice note %}} 
Do người dùng đã chọn được thể loại yêu thích nên tính năng **Gợi ý phim dựa trên lịch sử tương tác** sẽ đưa ra kết quả tương tự như **Gợi ý phim theo thể loại yêu thích**. 
{{% /notice %}}

## Returning user

{{% notice note %}}
Đối với người dùng đã đăng nhập, cũng có tính năng [**Gợi ý phim tương đồng theo phim mà người dùng nhập**](#gợi-ý-phim-tương-đồng-theo-phim-mà-người-dùng-nhập) như người dùng khách.
{{% /notice %}}

### Gợi ý phim dựa trên lịch sử tương tác

**Ví dụ:** `user_id = 1`. Trước hết, xem qua lịch sử tương tác `rating` có sẵn của người dùng này.

![return user 1](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/return-user-1.png)

Sau đó, dùng tính năng **Gợi ý phim dựa trên lịch sử tương tác** để gợi ý các bộ phim dựa trên lịch sử `rating` trên.

![return user 2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/return-user-2.png)

Danh sách phim gợi ý có nhiều thể loại phổ biến như `Action`, `Adventure`, `Fantasy`, `Family`. Kiểm tra nếu người dùng tương tác nhiều với các phim thuộc thể loại `Music`, `Romance` thì danh sách có thay đổi không.

Tiến hành nạp một tập dữ liệu giả lập bao gồm các tương tác mới nhất của người dùng này. Cụ thể, người dùng đã thực hiện các hành vi có trọng số cao `rating: 5.0`, `watch: 1.0` đối với các bộ phim thuộc thể loại `Music` và `Romance`, đồng thời thực hiện một lượt `click` vào bộ phim **Harry Potter and the Half-Blood Prince** (thuộc thể loại `Adventure`, `Fantasy`) đã từng xem trong quá khứ.

![return user 3](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/return-user-3.png)

Danh sách phim sau khi mô hình đưa ra gợi ý:

![return user 4](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/return-user-4.png)

- **Bảo toàn sở thích dài hạn:** Các vị trí Top 1-4 đều thuộc về chuỗi phim **Harry Potter**. Hệ thống nhận diện sự liên kết giữa cú click ngắn hạn hiện tại và lịch sử đánh giá 5 sao trong quá khứ, từ đó đẩy các phim cùng series lên đầu. Điều này chứng minh hệ thống không bị mất trí nhớ khi người dùng có sở thích mới.

- **Thỏa hiệp với tương tác ngắn hạn:** Tại các vị trí kế tiếp, mô hình bắt đầu chèn vào các bộ phim mang đặc trưng `Music` và `Romance`. Hệ thống đã thành công trong việc mở rộng không gian gợi ý, đáp ứng tức thời nhu cầu mới phát sinh của người dùng mà không cần chờ huấn luyện lại toàn bộ mô hình.

## Đánh giá mô hình

Để đo lường hiệu suất thực tế của các thuật toán gợi ý, quá trình kiểm thử định lượng được thực hiện trên tập dữ liệu `test`. Mỗi người dùng sẽ bị ẩn đi một bộ phim mà họ thực sự yêu thích, sau đó đối chiếu xem danh sách mô hình dự đoán có chứa bộ phim đó hay không.

### 1. Cấu hình kiểm thử

- **Tập đánh giá:** Mẫu 5.000 người dùng.
- **Ngưỡng yêu thích:** Phim bị giấu phải có điểm `rating >= 4.0`.
- **Kích thước danh sách gợi ý:** Top 20 phim.
- **Baseline:** Sử dụng mô hình `popularity_train` (gợi ý phim phổ biến nhất toàn hệ thống). Baseline được dựng lại hoàn toàn từ tập huấn luyện để tránh rò rỉ dữ liệu.



### 2. Kết quả đánh giá

Bảng dưới đây tổng hợp hiệu suất của 4 luồng mô hình đã được triển khai:

| Mô hình | HitRate@10 | NDCG@10 | Phim khác nhau | Độ phủ |
| :--- | :---: | :---: | :---: | :---: |
| **Popularity - Baseline** | 0.0332 | 0.0201 | 128 | 0.28% |
| **Collaborative - ALS** | 0.1115 | 0.0537 | 2.411 | 5.31% |
| **Content-based**| 0.0051 | 0.0035 | 17.530 | 38.59% |
| **Hybrid - RRF** | 0.0818 | 0.0393 | 8.110 | 17.85% |

### 3. Phân tích và Đánh giá

Dựa trên các chỉ số thu được, chúng ta có thể rút ra những đánh giá quan trọng về đặc tính của từng thuật toán:

- **Mô hình Popularity:** Chỉ đạt HitRate@10 ở mức 0.0332 và độ phủ vô cùng hạn hẹp (chỉ 0.28% với 128 phim được gợi ý ra). Lý do là mô hình này áp dụng chung một danh sách phim **Top-Rated** cho tất cả người dùng, dẫn đến tính cá nhân hóa bằng 0.
- **Mô hình Collaborative Filtering:** Đóng vai trò là thuật toán dự đoán chính xác nhất hệ thống. ALS vượt trội hoàn toàn so với Baseline khi đạt HitRate@10 là 0.1115 (tăng **+235.8%**) và NDCG@10 cao nhất (0.0537). Tuy nhiên, nhược điểm chí mạng của phương pháp này đã xuất hiện: Có 57 người dùng bị rơi vào trạng thái **Cold-start** nên hệ thống hoàn toàn không thể sinh ra gợi ý cho họhọ.
- **Mô hình Content-based:** Phương pháp này sử dụng 3 bộ phim đánh giá cao sớm nhất của người dùng làm mỏ neo để tìm phim tương đồng. Dù có độ chính xác thấp nhất (kém Baseline 84.8%), nó lại mang đến không gian khám phá khổng lồ với độ phủ lên tới **38.59%** (17.530 bộ phim khác nhau).
- **Mô hình Hybrid - Kết hợp bằng Weighted RRF:** Đây là thành quả tối ưu nhất của hệ thống, minh chứng cho sự kết hợp hài hòa giữa các thuật toán. Mô hình lai giữ được độ chính xác rất cao, với HitRate@10 đạt 0.0818 (vượt Baseline **+146.4%**). Đáng chú ý, thuật toán kết hợp đã xử lý thành công toàn bộ 57 người dùng bị lỗi "Cold-start" của mạng ALS thông qua tầng Fallback toàn cục, đồng thời kéo độ phủ của hệ thống lên mức an toàn và đa dạng hóa danh mục lên 8.110 phim (17.85%).

**Kết luận:** Mô hình Hybrid đã hoàn thành xuất sắc mục tiêu thiết kế, đó là duy trì độ chính xác cao từ dữ liệu quá quá khứ, đảm bảo tính đa dạng của nội dung và giải quyết triệt để sự cố không có dữ liệu quá khứ.