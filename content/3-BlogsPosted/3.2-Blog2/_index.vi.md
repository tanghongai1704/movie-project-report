---
title: "Blog 2"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# TÌM HIỂU AMAZON MACIE – TỰ ĐỘNG PHÁT HIỆN DỮ LIỆU NHẠY CẢM TRONG AMAZON S3

Trong quá trình tìm hiểu về các dịch vụ bảo mật trên AWS, mình có biết đến Amazon Macie. Ban đầu mình nghĩ đây chỉ là một công cụ để kiểm tra cấu hình của S3 Bucket, nhưng sau khi đọc tài liệu thì mình nhận ra Macie có khả năng làm nhiều hơn thế.

Amazon Macie là một dịch vụ sử dụng Machine Learning và Pattern Matching để tự động phát hiện dữ liệu nhạy cảm được lưu trữ trong Amazon S3. Thay vì phải mở từng file để kiểm tra, Macie có thể quét dữ liệu và cảnh báo nếu phát hiện những thông tin như số thẻ tín dụng, địa chỉ email, số điện thoại hoặc các thông tin nhận dạng cá nhân (PII).

Theo mình, đây là một service khá hữu ích trong các hệ thống lưu trữ nhiều tài liệu hoặc dữ liệu người dùng.

## Amazon Macie có thể làm gì?

Sau khi tìm hiểu, mình thấy Macie hỗ trợ khá nhiều chức năng như:

- Phát hiện dữ liệu nhạy cảm trong Amazon S3.
- Xác định các S3 Bucket đang chứa dữ liệu quan trọng.
- Kiểm tra Bucket có đang public hoặc được chia sẻ ngoài mong muốn hay không.
- Đưa ra mức độ rủi ro để người quản trị ưu tiên xử lý.
- Tạo báo cáo về kết quả quét để theo dõi và kiểm tra.

{{% notice tip %}}
Điểm mình thấy hay là Macie có thể hoạt động liên tục và cập nhật kết quả khi dữ liệu trong S3 thay đổi.
{{% /notice %}}

## Thử sử dụng Amazon Macie

Để hiểu rõ hơn, mình thử tìm hiểu quy trình quét một S3 Bucket bằng Amazon Macie.

**Bước 1:** Truy cập AWS Console và tìm Amazon Macie.

Đăng nhập AWS Management Console và tìm: Amazon Macie.

**Bước 2:** Chọn Enable Macie.

Sau khi kích hoạt, Macie sẽ bắt đầu thu thập thông tin về các S3 Bucket trong tài khoản AWS.

**Bước 3:** Chọn Create Job.

AWS cho phép tạo một công việc quét dữ liệu theo yêu cầu hoặc theo lịch định kỳ.

**Bước 4:** Chọn S3 Bucket cần quét.

Có thể chọn một hoặc nhiều Bucket tùy theo nhu cầu.

**Bước 5:** Chọn loại dữ liệu cần phát hiện.

Macie đã cung cấp sẵn nhiều loại dữ liệu nhạy cảm như:

```text
Email Address
Credit Card Number
Phone Number
Passport Number
Personal Information
```

Ngoài ra, người dùng cũng có thể tạo các tiêu chí phát hiện riêng cho từng trường hợp.

**Bước 6:** Bắt đầu quá trình quét.

Sau khi hoàn tất, Macie sẽ hiển thị kết quả trên Dashboard.

Nếu phát hiện dữ liệu nhạy cảm, kết quả sẽ bao gồm:
- Loại dữ liệu được phát hiện.
- Bucket chứa dữ liệu.
- Mức độ nghiêm trọng.
- Thời gian phát hiện.

## Những điểm mình thấy hữu ích

Sau khi tìm hiểu, mình thấy Amazon Macie có một số ưu điểm như:

- Tự động phát hiện dữ liệu nhạy cảm mà không cần kiểm tra thủ công.
- Dashboard trực quan, dễ theo dõi kết quả quét.
- Hỗ trợ nhiều loại dữ liệu nhạy cảm được định nghĩa sẵn.
- Có thể kết hợp với Amazon EventBridge hoặc Security Hub để xây dựng quy trình cảnh báo tự động.
- Giúp doanh nghiệp dễ dàng kiểm tra việc lưu trữ dữ liệu có tuân thủ các yêu cầu bảo mật hay không.

{{% notice note %}}
Theo mình, đây là một công cụ phù hợp với các hệ thống lưu trữ dữ liệu khách hàng hoặc tài liệu nội bộ.
{{% /notice %}}

## Một số điểm cần lưu ý

Bên cạnh những ưu điểm trên, mình cũng thấy có một vài điểm cần cân nhắc:

- Amazon Macie hiện chỉ tập trung vào dữ liệu lưu trữ trên Amazon S3, vì vậy nếu dữ liệu nằm ở các dịch vụ khác thì cần sử dụng thêm các giải pháp phù hợp.
- Ngoài ra, khi quét các Bucket có dung lượng lớn hoặc số lượng đối tượng nhiều, chi phí sử dụng cũng sẽ tăng theo. Do đó, nên lựa chọn phạm vi quét phù hợp thay vì quét toàn bộ dữ liệu nếu không cần thiết.
- Cuối cùng, Macie chỉ giúp phát hiện dữ liệu nhạy cảm chứ không tự động mã hóa hoặc xóa dữ liệu. Người quản trị vẫn cần đánh giá kết quả và thực hiện các biện pháp bảo vệ phù hợp.

## Khi nào nên sử dụng?

Theo mình, Amazon Macie sẽ phù hợp trong các trường hợp như:

- Kiểm tra S3 có chứa dữ liệu nhạy cảm hay không.
- Xác định các Bucket có nguy cơ rò rỉ dữ liệu.
- Hỗ trợ kiểm tra trước khi chia sẻ dữ liệu cho đối tác.
- Theo dõi việc lưu trữ dữ liệu khách hàng trong các hệ thống sử dụng Amazon S3.

## Kết luận

Sau khi tìm hiểu, mình thấy Amazon Macie là một service khá thú vị vì kết hợp giữa bảo mật và Machine Learning để hỗ trợ quản lý dữ liệu. Thay vì kiểm tra thủ công từng file, Macie giúp phát hiện nhanh những dữ liệu cần được bảo vệ, từ đó giảm rủi ro rò rỉ thông tin trong quá trình vận hành hệ thống.

Mình nghĩ đây là một service đáng để tìm hiểu nếu sau này làm việc với các ứng dụng lưu trữ nhiều dữ liệu trên Amazon S3 hoặc quan tâm đến các chủ đề về bảo mật dữ liệu.

Nếu anh/chị hoặc các bạn đã từng sử dụng Amazon Macie trong thực tế, rất mong được nghe thêm kinh nghiệm hoặc các tình huống sử dụng khác để cùng trao đổi.

## Tài liệu tham khảo

1. [AWS Documentation – Amazon Macie:](https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html)

2. [Getting Started with Amazon Macie:](https://docs.aws.amazon.com/.../user/getting-started.html)

3. [Amazon Macie Features:](https://aws.amazon.com/macie/features/)

4. [Amazon Macie Pricing:](https://aws.amazon.com/macie/pricing/)