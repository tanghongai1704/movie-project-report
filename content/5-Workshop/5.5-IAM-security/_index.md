---
title: "IAM, Least Privilege, and Security"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

The repository uses boto3's default credential provider chain:

- Developers should use AWS IAM Identity Center or a profile.
- EC2 uses an instance profile.
- SageMaker uses an execution role.

The exact role names, JSON policies, trust relationships, and ARNs are not stored in the repository.

![Credential flow between developers, EC2, and AWS services](/images/5-Workshop/5.5-IAM-security/security-credential-flow.png)

*A developer profile or EC2 instance profile provides credentials to boto3; the JWT secret is managed separately for FastAPI authentication.*

## 1. Secure the AWS Account

An AWS account contains IAM identities; there is no separate “IAM account” for each user. Bootstrap the account as follows:

1. Sign in as the root user only for operations that require account-level access.
2. Enable MFA for the root user and do not create a root access key.
3. Create an IAM Identity Center user for the operator. If a personal workshop does not use Identity Center, create an IAM user as described in section 2.
4. Enable IAM access to Billing once if an IAM user/role needs to configure AWS Budgets; then sign out of the root user.
5. Use an IAM user/role for all remaining S3, DynamoDB, SageMaker, EC2, CloudWatch, and Budgets steps.

{{% notice warning %}}
Do not use the root user for daily workshop activities, and do not share one IAM user among multiple people.
{{% /notice %}}

## 2. Create an IAM Group and IAM User

Use this process only when IAM Identity Center has not been deployed.

### 2.1. Create the Group

1. Open **IAM** → **User groups** → **Create group**.
2. Enter the name `movie-recommendation-workshop-operators`.
3. Under permissions, select the customer-managed policy `<WORKSHOP_OPERATOR_POLICY>` reviewed according to section 3.
4. Select **Create group**.

### 2.2. Create the User and Add It to the Group

1. Open **IAM** → **Users** → **Create user**.
2. Enter a username identifying the individual operator; do not use a generic name such as `admin`.
3. Enable Console access only if the user needs the AWS Management Console.
4. Add the user to `movie-recommendation-workshop-operators`.
5. Complete user creation, sign in as the new user, and enable MFA.
6. Create an access key only when the AWS CLI is actually required. Do not place the access key in `.env`, source code, GitHub Actions logs, or screenshots.

<!-- IMAGE-5.5-IAM-01: IAM user in the workshop operator group with MFA Enabled. -->

## 3. Create and Attach a Permission Policy

1. Open **IAM** → **Policies** → **Create policy**.
2. Select the **JSON** tab or visual editor and add only the actions required for the workshop.
3. Restrict S3, DynamoDB, SageMaker, and IAM PassRole by Region, resource ARN, and tag where the service supports it.
4. Name the policy `<WORKSHOP_OPERATOR_POLICY>` and add a description/owner.
5. Select **Create policy**.
6. Return to **User groups** → `movie-recommendation-workshop-operators` → **Permissions** → **Add permissions** → **Attach policies**.
7. Select `<WORKSHOP_OPERATOR_POLICY>` → **Add permissions**.

The operator policy should be built by the security owner from the following permission groups:

| Operation group | Permissions to grant within scope |
|---|---|
| S3 setup | Create a bucket; configure ownership, Block Public Access, versioning, encryption, and tags; perform object operations on the correct bucket/prefixes |
| DynamoDB setup | Create/describe five tables; configure TTL, PITR, encryption/tags; read/write workshop data |
| SageMaker setup | Create/describe Processing Jobs, Models, EndpointConfigs, and Endpoints; invoke the endpoint; view logs/metrics |
| EC2 setup | Launch/describe instances; create/attach security groups; manage rules and tags; attach the correct instance profile |
| CloudWatch | Create log groups, retention settings, metric filters, and alarms; read workshop logs/metrics |
| Budgets | View/create/edit the correct budget and configure approved notifications |
| PassRole | Grant `iam:PassRole` only for the designated SageMaker execution role and EC2 role |

{{% notice warning %}}
Do not attach `AdministratorAccess` or `IAMFullAccess` to bypass permission errors. If an AWS-managed FullAccess policy is used in an isolated lab account, document it as a temporary exception, set a removal deadline, and do not reuse it in production.
{{% /notice %}}

<!-- IMAGE-5.5-IAM-02: Customer-managed policy attached to the operator group. -->

## 4. Create Service Roles for SageMaker and EC2

### 4.1. SageMaker Execution Role

1. Open **IAM** → **Roles** → **Create role**.
2. Select **AWS service** → **SageMaker** as the trusted service.
3. Attach a customer-managed policy that permits reading/writing only the correct S3 input/output prefixes, pulling the correct ECR image, and writing CloudWatch Logs.
4. If the Processing Job runs in a VPC, add the network interface management permissions described in the SageMaker documentation.
5. Name the role `movie-rec-sagemaker-execution-role` and create it.
6. Verify that the trust relationship allows only `sagemaker.amazonaws.com` to assume the role.

### 4.2. EC2 Instance Role

1. Open **IAM** → **Roles** → **Create role**.
2. Select **AWS service** → **EC2**.
3. Attach a backend runtime policy restricted to the five DynamoDB tables, the S3 bucket/prefixes, and one SageMaker Endpoint.
4. Attach `CloudWatchAgentServerPolicy` or an equivalent customer-managed policy to send metrics/logs.
5. If the agent is installed through Systems Manager, add the minimum SSM permissions approved by the platform team.
6. Name the role `movie-rec-ec2-application-role`, create it, and select this role in the IAM instance profile when launching EC2.

<!-- IMAGE-5.5-IAM-03: Trust relationship and permission policies of the SageMaker execution role. -->

<!-- IMAGE-5.5-IAM-04: EC2 role/instance profile with backend runtime and CloudWatch Agent permissions. -->

## 5. Principal and Permission Matrix

| Principal | Required permissions | Resource scope |
|---|---|---|
| Backend EC2 role | `sts:GetCallerIdentity` | Current identity |
| Backend EC2 role | DynamoDB describe/get/batch-get/put/query/scan according to code paths | ARNs of the five tables and related indexes |
| Backend EC2 role | S3 list for startup validation | One bucket, restricted by prefix where possible |
| Backend EC2 role | `sagemaker:DescribeEndpoint`, `sagemaker:InvokeEndpoint` | One endpoint |
| S3/ML operator | S3 list/get/put | Data, model, and report prefixes |
| Processing submitter | Create/describe Processing Jobs, `iam:PassRole` | Job namespace and one execution role |
| SageMaker execution role | S3 list/get/put, CloudWatch Logs | Specific prefixes and log groups |
| GitHub deployer | SSH credential stored in GitHub Secrets | One EC2 host |

{{% notice warning %}}
Do not use `Action: "*"` or `Resource: "*"` merely to make the system work. Production policies must be reviewed by the security owner.
{{% /notice %}}

## 6. Least-Privilege Observations

- The backend HTTP runtime does not require `DeleteItem` because there is currently no delete API.
- S3 upload/download permissions should belong to a separate tooling role instead of expanding the web runtime role.
- `Users` currently requires a scan for case-insensitive login/uniqueness because there is no identity GSI.
- The SageMaker deployment principal and the SageMaker execution role are two different principals.
- `iam:PassRole` should be permitted only for the correct execution role.

## 7. Application Security

- Passwords use PBKDF2-HMAC-SHA256 with a random salt and a configured iteration count.
- HS256 JWTs validate the signature, issuer, audience, and expiry.
- A guest attempting a protected action receives `401`.
- A user who has not completed onboarding receives `403` when requesting recommendations.
- The frontend stores the access token in `localStorage`, so XSS must be controlled.
- Logout currently does not revoke the JWT on the server; the endpoint only returns `204`.

## 8. Positive Test

With the correct backend role attached:

```bash
aws sts get-caller-identity \
  --region "<AWS_REGION>"

aws dynamodb describe-table \
  --table-name "<AUTHORIZED_TABLE_NAME>" \
  --region "<AWS_REGION>"

aws sagemaker describe-endpoint \
  --endpoint-name "<SAGEMAKER_ENDPOINT_NAME>" \
  --region "<AWS_REGION>"
```

Pass criterion: all three approved operations succeed.

{{% notice note %}}
`sts:GetCallerIdentity` has special behavior and is not sufficient to prove that the principal can access DynamoDB, S3, or SageMaker.
{{% /notice %}}

<!-- IMAGE-5.5-01: IAM role with a resource-restricted policy; ARN/account ID redacted. -->

## 9. Safe Negative Test

The security owner must provide a dedicated resource outside the approved scope for testing:

```bash
aws dynamodb describe-table \
  --table-name "<APPROVED_OUT_OF_SCOPE_TEST_TABLE>" \
  --region "<AWS_REGION>"
```

Expected result: `AccessDeniedException`.

`ResourceNotFoundException` does not prove least privilege because the resource might not exist.

<!-- IMAGE-5.5-02: AccessDenied when accessing an out-of-scope test resource. -->

## 10. Configure Amazon CloudWatch

### 10.1. Create Log Groups and Set Retention

1. Open **CloudWatch** → **Logs** → **Log groups** → **Create log group**.
2. Create environment-specific log groups, for example:
   - `/movie-rec/<ENVIRONMENT>/system`
   - `/movie-rec/<ENVIRONMENT>/backend`
   - `/movie-rec/<ENVIRONMENT>/frontend`
3. Select a KMS key if policy requires encryption with a customer-managed key.
4. After creation, select the log group → **Actions** → **Edit retention setting**.
5. Set appropriate retention, such as 14 or 30 days for the workshop; do not leave `Never expire` unless retention is required.

SageMaker automatically creates log groups/streams for Processing Jobs and Endpoints when the execution role has permission. Do not rename service-managed log groups; use tags, dashboards, and alarms to centralize observability.

![CloudWatch log groups for SageMaker Endpoints and Processing Jobs](/images/5-Workshop/5.5-IAM-security/cloudwatch-sagemaker-log-groups.png)

### 10.2. Install the CloudWatch Agent on EC2

1. Confirm that the EC2 instance profile has `CloudWatchAgentServerPolicy` or an equivalent least-privilege policy.
2. Open **CloudWatch** → **Getting Started** → the **Install and configure CloudWatch agent** workflow.
3. Select the EC2 instance by ID or the `Project=movie-recommendation` tag.
4. Select a suitable workload/configuration and collect at least memory, disk usage, and the required system metrics.
5. Specify the actual application and operating-system log files. For Docker stdout/stderr, configure a logging driver or log-forwarding mechanism before defining the destination log group.
6. Map logs to the correct log group and a stream name containing the instance ID/container name.
7. Apply the configuration, wait until the agent is running, and confirm that the log stream receives new events.

If the Console option for installing the agent is unavailable in the Region, use AWS Systems Manager or install it manually according to the CloudWatch Agent documentation.

<!-- IMAGE-5.5-CLOUDWATCH-01: CloudWatch Agent configuration and log groups with log streams from EC2/backend. -->

### 10.3. Create Alarms

1. Open **CloudWatch** → **Alarms** → **All alarms** → **Create alarm**.
2. Select **Select metric**, then choose the service/resource and metric to monitor.
3. Select the statistic, period, threshold, and number of breaching datapoints.
4. Under **Notification**, select or create an SNS topic and enter the alert recipient's email address.
5. Confirm the subscription when using SNS email.
6. Give the alarm a name containing the environment and resource, review it, and select **Create alarm**.

| Resource | Suggested metric | Example workshop condition |
|---|---|---|
| EC2 | `CPUUtilization` | Greater than 80% for multiple consecutive datapoints |
| EC2 | `StatusCheckFailed` | Greater than or equal to 1 |
| DynamoDB | `ThrottledRequests` and `SystemErrors` | Greater than or equal to 1 during the evaluation period |
| SageMaker Endpoint | `Invocation5XXErrors` | Greater than or equal to 1 |
| SageMaker Endpoint | `ModelLatency` | Monitor the percentile and set a threshold after establishing a baseline |

These thresholds are only starting points for the workshop, not production SLAs.

<!-- IMAGE-5.5-CLOUDWATCH-02: CloudWatch alarm in the OK state with an SNS notification configured. -->

## 11. Configure AWS Budgets

### 11.1. Allow IAM Access to Billing

1. Sign in as the root user for this one-time operation.
2. Open **Billing and Cost Management** → the account/billing access settings.
3. Under **IAM User and Role Access to Billing Information**, select **Edit** → enable **Activate IAM Access** → save the change.
4. Sign out of the root user immediately after completion.
5. Attach a least-privilege Budgets/Billing policy to the operator role or group. Enabling IAM access does not grant permissions automatically.

### 11.2. Create a Monthly Cost Budget

1. Open **Billing and Cost Management** → **Budgets** → **Create budget**.
2. Select **Customize (advanced)** → **Cost budget** → **Next**.
3. Enter a budget name, such as `movie-recommendation-workshop-monthly`.
4. Select the **Monthly** period and a recurring budget, and enter `<MONTHLY_BUDGET_USD>`.
5. Under scope/filter, restrict by AWS service, account, or the `Project=movie-recommendation` cost allocation tag after the tag has been activated for billing.
6. Create these example alerts:
   - Actual cost reaches 50%.
   - Actual cost reaches 80%.
   - Forecasted cost reaches 100%.
7. Add only the email addresses that must receive notifications; an SNS topic can also be added if the organization uses a centralized alert channel.
8. For the first workshop, do not enable an automatic budget action that stops or deletes resources without an approved runbook.
9. Review → **Create budget**, and verify the notification status.

![AWS Budget tracking workshop costs](/images/5-Workshop/5.5-IAM-security/aws-budget-overview.png)

*Billing and Cost Management confirms that the `My-200$-budget` budget has a USD 200 limit, an `OK` threshold state, and a `Healthy` health status.*

<!-- IMAGE-5.5-BUDGETS-01: Monthly cost budget with the amount, scope, and three alert thresholds. -->
