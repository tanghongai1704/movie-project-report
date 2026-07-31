---
title: "Blog 2"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# EXPLORING AMAZON MACIE – AUTOMATICALLY DISCOVER SENSITIVE DATA IN AMAZON S3

While exploring security services on AWS, I came across Amazon Macie. Initially, I thought it was just a tool to check S3 Bucket configurations, but after reading the documentation, I realized Macie can do much more than that.

Amazon Macie is a service that uses Machine Learning and Pattern Matching to automatically discover sensitive data stored in Amazon S3. Instead of having to open each file to check, Macie can scan data and alert if it detects information such as credit card numbers, email addresses, phone numbers, or personally identifiable information (PII).

In my opinion, this is a quite useful service in systems that store a lot of documents or user data.

## What can Amazon Macie do?

After researching, I found that Macie supports quite a few features such as:

- Discovering sensitive data in Amazon S3.
- Identifying S3 Buckets containing important data.
- Checking whether Buckets are public or shared unintentionally.
- Providing risk levels for administrators to prioritize handling.
- Generating reports on scan results for tracking and inspection.

{{% notice tip %}}
What I find interesting is that Macie can operate continuously and update results when data in S3 changes.
{{% /notice %}}

## Trying out Amazon Macie

To better understand, I looked into the process of scanning an S3 Bucket using Amazon Macie.

**Step 1:** Access the AWS Console and search for Amazon Macie.

Log in to the AWS Management Console and search for: Amazon Macie.

**Step 2:** Select Enable Macie.

Once enabled, Macie will start collecting information about S3 Buckets in the AWS account.

**Step 3:** Select Create Job.

AWS allows creating a data scanning job on demand or on a recurring schedule.

**Step 4:** Select the S3 Bucket to scan.

You can select one or multiple Buckets depending on your needs.

**Step 5:** Select the types of data to discover.

Macie provides many pre-defined sensitive data types such as:

```text
Email Address
Credit Card Number
Phone Number
Passport Number
Personal Information
```

Additionally, users can also create custom detection criteria for specific cases.

**Step 6:** Start the scanning process.

Once completed, Macie will display the results on the Dashboard.

If sensitive data is detected, the results will include:
- The type of data detected.
- The bucket containing the data.
- The severity level.
- The detection timestamp.

## What I find useful

After researching, I found that Amazon Macie has several advantages such as:

- Automatically detecting sensitive data without manual checking.
- Intuitive dashboard, making it easy to track scan results.
- Supporting many pre-defined sensitive data types.
- Integrating with Amazon EventBridge or Security Hub to build automated notification workflows.
- Helping businesses easily check whether data storage complies with security requirements.

{{% notice note %}}
In my opinion, this is a tool suitable for systems storing customer data or internal documents.
{{% /notice %}}

## Some points to keep in mind

Besides the above advantages, I also noticed a few points to consider:

- Amazon Macie currently focuses only on data stored in Amazon S3, so if data resides in other services, appropriate additional solutions are needed.
- Additionally, when scanning Buckets with large sizes or high object counts, usage costs will also increase accordingly. Therefore, it is advisable to choose an appropriate scan scope instead of scanning all data unnecessarily.
- Finally, Macie only helps detect sensitive data and does not automatically encrypt or delete data. Administrators still need to evaluate the results and take appropriate protective measures.

## When should it be used?

In my opinion, Amazon Macie will be suitable in cases such as:

- Checking whether S3 contains sensitive data.
- Identifying Buckets at risk of data leakage.
- Assisting with checks before sharing data with partners.
- Monitoring customer data storage in systems using Amazon S3.

## Conclusion

After researching, I found Amazon Macie to be a quite interesting service because it combines security and Machine Learning to support data management. Instead of manually checking each file, Macie helps quickly detect data that needs protection, thereby reducing the risk of information leakage during system operations.

I think this is a service worth exploring if you work with data-heavy applications on Amazon S3 in the future or care about data security topics.

If anyone has used Amazon Macie in practice, I would love to hear more about your experiences or other use cases to exchange ideas.

## References

1. [AWS Documentation – Amazon Macie:](https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html)

2. [Getting Started with Amazon Macie:](https://docs.aws.amazon.com/.../user/getting-started.html)

3. [Amazon Macie Features:](https://aws.amazon.com/macie/features/)

4. [Amazon Macie Pricing:](https://aws.amazon.com/macie/pricing/)