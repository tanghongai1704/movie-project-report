---
title: "Blog 3"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# TÌM HIỂU AMAZON SIMPLE EMAIL SERVICE (AMAZON SES) – DỊCH VỤ GỬI EMAIL TRÊN AWS

Trong quá trình tìm hiểu các dịch vụ của AWS, mình có dịp đọc về Amazon Simple Email Service (Amazon SES). Ban đầu mình chỉ nghĩ nếu muốn gửi email từ ứng dụng thì có thể dùng Gmail SMTP hoặc một số dịch vụ như SendGrid. Tuy nhiên, khi tìm hiểu thêm thì mình biết AWS cũng có một dịch vụ chuyên cho việc gửi và nhận email với tên gọi là Amazon SES.

Theo tài liệu của AWS, SES được thiết kế để hỗ trợ các ứng dụng gửi email với quy mô từ nhỏ đến lớn. Một số trường hợp phổ biến có thể kể đến như gửi email xác thực tài khoản, OTP, thông báo đơn hàng, email marketing hoặc newsletter.

Điều mình thấy thú vị là SES không chỉ hỗ trợ giao thức SMTP mà còn cung cấp API để tích hợp trực tiếp vào ứng dụng.

## Một ví dụ đơn giản

Để hiểu rõ hơn, mình thử tìm hiểu quy trình gửi email bằng Amazon SES.

**Bước 1:** Truy cập AWS Console và tìm Amazon SES.

**Bước 2:** Chọn Region mà SES hỗ trợ.

{{% notice note %}}
Lưu ý rằng không phải Region nào cũng hỗ trợ đầy đủ các tính năng của SES.
{{% /notice %}}

**Bước 3:** Xác minh địa chỉ email.

Trong môi trường Sandbox, AWS yêu cầu xác minh email gửi và email nhận.

Chỉ cần nhập địa chỉ email, AWS sẽ gửi một email xác nhận. Sau khi nhấn vào liên kết xác minh thì email sẽ được kích hoạt.

**Bước 4:** Thử gửi email.

Có thể chọn mục Test Email ngay trên AWS Console hoặc sử dụng SMTP/API để gửi từ ứng dụng.

Ví dụ nội dung email:

```text
Subject: Welcome
Body: Welcome to our application.
```

**Bước 5:** Kiểm tra hộp thư nhận.

Nếu mọi thứ được cấu hình đúng, email sẽ được gửi đến địa chỉ đã xác minh.

## Một vài điểm mình thấy hữu ích

Sau khi tìm hiểu, mình thấy Amazon SES có một số ưu điểm như:

- Dễ tích hợp thông qua SMTP hoặc AWS SDK.
- Có khả năng gửi số lượng email lớn khi ứng dụng phát triển.
- Theo dõi được tỷ lệ gửi thành công, bounce và complaint.
- Chi phí tương đối thấp so với nhiều dịch vụ gửi email khác.
- Có thể kết hợp với Lambda, SNS hoặc EventBridge để xây dựng quy trình xử lý email tự động.

{{% notice tip %}}
Theo mình, đây là một service khá phù hợp nếu đang xây dựng các ứng dụng web hoặc mobile cần gửi email cho người dùng.
{{% /notice %}}

## Một số điểm cần lưu ý

Bên cạnh những ưu điểm trên thì cũng có một số điều mình thấy cần quan tâm.

Khi mới tạo tài khoản, SES sẽ hoạt động ở Sandbox Mode. Điều này có nghĩa là chỉ có thể gửi email đến những địa chỉ đã được xác minh trước.

Nếu muốn gửi email đến người dùng thực tế thì cần gửi yêu cầu lên AWS để chuyển sang Production Access.

Ngoài ra, nếu nội dung email không được thiết kế hợp lý hoặc gửi quá nhiều email trong thời gian ngắn thì vẫn có khả năng bị đánh dấu là spam giống như các nền tảng gửi email khác.

## Khi nào nên sử dụng?

Theo mình, Amazon SES sẽ phù hợp với các trường hợp như:

- Gửi email xác thực tài khoản.
- Gửi mã OTP.
- Quên mật khẩu (Forgot Password).
- Gửi hóa đơn hoặc thông báo đơn hàng.
- Gửi email định kỳ cho khách hàng.
- Gửi thông báo từ hệ thống.

Đây đều là những tính năng khá phổ biến trong hầu hết các ứng dụng hiện nay.

## Kết luận

Sau khi tìm hiểu, mình thấy Amazon SES là một dịch vụ khá hữu ích nhưng thường ít được nhắc đến khi mới bắt đầu học AWS. Việc tích hợp không quá phức tạp, chi phí hợp lý và có thể mở rộng khi hệ thống có nhiều người dùng hơn.

Mình nghĩ đây là một service đáng để thử nếu sau này xây dựng các dự án có chức năng gửi email, thay vì phải tự thiết lập mail server hoặc phụ thuộc hoàn toàn vào SMTP của Gmail.

Nếu anh/chị hoặc các bạn đã từng sử dụng Amazon SES trong thực tế thì rất mong được nghe thêm kinh nghiệm hoặc những lưu ý khi triển khai để cùng học hỏi.

## Tài liệu tham khảo
1. [AWS Documentation – Amazon Simple Email Service (SES)](https://docs.aws.amazon.com/ses/latest/dg/Welcome.html)
2. [Getting Started with Amazon SES](https://docs.aws.amazon.com/.../dg/getting-started.html)
3. [Amazon SES Pricing](https://aws.amazon.com/ses/pricing/)
4. [AWS Messaging Blog – Amazon SES](https://aws.amazon.com/blogs/messaging-and-targeting/)