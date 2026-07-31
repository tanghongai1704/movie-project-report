---
title: "Blog 3"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# EXPLORING AMAZON SIMPLE EMAIL SERVICE (AMAZON SES) – AWS EMAIL SENDING SERVICE

While learning about AWS services, I had the opportunity to read about Amazon Simple Email Service (Amazon SES). Initially, I thought that if I wanted to send emails from an application, I could use Gmail SMTP or a service like SendGrid. However, upon further research, I learned that AWS also has a dedicated service for sending and receiving emails called Amazon SES.

According to AWS documentation, SES is designed to support applications sending emails on a scale from small to large. Some common use cases include sending account verification emails, OTPs, order notifications, email marketing, or newsletters.

What I found interesting is that SES not only supports the SMTP protocol but also provides APIs to integrate directly into applications.

## A Simple Example

To better understand, I tried exploring the process of sending emails using Amazon SES.

**Step 1:** Access the AWS Console and search for Amazon SES.

**Step 2:** Choose the Region supported by SES.

{{% notice note %}}
Note that not all Regions fully support all features of SES.
{{% /notice %}}

**Step 3:** Verify your email address.

In the Sandbox environment, AWS requires verifying both the sender and recipient email addresses.

Just enter the email address, and AWS will send a confirmation email. After clicking the verification link, the email will be activated.

**Step 4:** Try sending an email.

You can select the Test Email option directly in the AWS Console or use SMTP/API to send from your application.

Ví dụ nội dung email:

```text
Subject: Welcome
Body: Welcome to our application.
```

**Step 5:** Check your inbox.

If everything is configured correctly, the email will be delivered to the verified address.

## A Few Useful Points

After exploring, I found that Amazon SES has several advantages, such as:

- Easy integration via SMTP or AWS SDK.
- Ability to send large volumes of emails as the application grows.
- Tracking success rates, bounces, and complaints.
- Relatively low cost compared to many other email sending services.
- Can be integrated with Lambda, SNS, or EventBridge to build automated email processing workflows.

{{% notice tip %}}
In my opinion, this is a quite suitable service if you are building web or mobile applications that need to send emails to users.
{{% /notice %}}

## Points to Keep in Mind

Besides the above advantages, there are also a few things I found important to consider.

When first creating an account, SES operates in Sandbox Mode. This means you can only send emails to pre-verified addresses.

If you want to send emails to real users, you need to submit a request to AWS to move to Production Access.

Additionally, if the email content is not properly designed or if too many emails are sent in a short period, there is still a risk of being marked as spam, just like on other email platforms.

## When Should You Use It?

In my opinion, Amazon SES is suitable for use cases such as:

- Sending account verification emails.
- Sending OTP codes.
- Password recovery (Forgot Password).
- Sending invoices or order notifications.
- Sending periodic customer newsletters.
- Sending system notifications.

These are quite common features in almost all modern applications.

## Conclusion

After exploring, I found that Amazon SES is a very useful service, though it is often overlooked when starting to learn AWS. Integration is not overly complex, costs are reasonable, and it can scale as the system grows to more users.

I think this is a service worth trying if you build projects with email functionality in the future, instead of having to set up your own mail server or rely entirely on Gmail SMTP.

If any of you have used Amazon SES in practice, I would love to hear your experiences or deployment tips to learn together.

## References
1. [AWS Documentation – Amazon Simple Email Service (SES)](https://docs.aws.amazon.com/ses/latest/dg/Welcome.html)
2. [Getting Started with Amazon SES](https://docs.aws.amazon.com/.../dg/getting-started.html)
3. [Amazon SES Pricing](https://aws.amazon.com/ses/pricing/)
4. [AWS Messaging Blog – Amazon SES](https://aws.amazon.com/blogs/messaging-and-targeting/)