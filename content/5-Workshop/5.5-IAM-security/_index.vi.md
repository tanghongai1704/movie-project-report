---
title: "IAM, nguyên tắc đặc quyền tối thiểu và bảo mật"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

Repository sử dụng default credential provider chain của boto3:

- Developer nên dùng AWS IAM Identity Center hoặc profile.
- EC2 sử dụng instance profile.
- SageMaker sử dụng execution role.

Exact role name, JSON policy, trust relationship và ARN chưa được lưu trong repository.

![Luồng credential giữa developer, EC2 và các dịch vụ AWS](/images/5-Workshop/5.5-IAM-security/security-credential-flow.png)

*Developer profile hoặc EC2 instance profile cung cấp credential cho boto3; JWT secret được quản lý riêng cho FastAPI authentication.*

## 1. Bảo vệ AWS account

AWS account chứa IAM identities; không có khái niệm “IAM account” riêng cho từng người dùng. Thực hiện bootstrap account như sau:

1. Đăng nhập root user chỉ cho thao tác bắt buộc ở cấp account.
2. Bật MFA cho root user và không tạo root access key.
3. Tạo IAM Identity Center user cho người vận hành. Nếu workshop cá nhân chưa dùng Identity Center, tạo IAM user theo mục 2.
4. Bật IAM access tới Billing một lần nếu IAM user/role cần cấu hình AWS Budgets; sau đó đăng xuất root.
5. Dùng IAM user/role cho toàn bộ bước S3, DynamoDB, SageMaker, EC2, CloudWatch và Budgets còn lại.

{{% notice warning %}}
Không dùng root user cho workshop hằng ngày và không chia sẻ một IAM user cho nhiều người.
{{% /notice %}}

## 2. Tạo IAM group và IAM user

Chỉ dùng quy trình này khi chưa triển khai IAM Identity Center.

### 2.1. Tạo group

1. Mở **IAM** → **User groups** → **Create group**.
2. Nhập tên `movie-recommendation-workshop-operators`.
3. Tại phần permissions, chọn customer managed policy `<WORKSHOP_OPERATOR_POLICY>` đã được review theo mục 3.
4. Chọn **Create group**.

### 2.2. Tạo user và thêm vào group

1. Vào **IAM** → **Users** → **Create user**.
2. Nhập username theo người thực hiện, không dùng tên chung như `admin`.
3. Chỉ bật Console access nếu user cần AWS Management Console.
4. Thêm user vào `movie-recommendation-workshop-operators`.
5. Hoàn tất tạo user, đăng nhập bằng user mới và bật MFA.
6. Chỉ tạo access key khi thực sự dùng AWS CLI. Không đặt access key trong `.env`, source code, GitHub Actions log hoặc ảnh chụp.

<!-- IMAGE-5.5-IAM-01: IAM user nằm trong workshop operator group và MFA ở trạng thái Enabled. -->

## 3. Tạo và gắn permission policy

1. Mở **IAM** → **Policies** → **Create policy**.
2. Chọn tab **JSON** hoặc visual editor và thêm đúng actions cần cho workshop.
3. Giới hạn S3, DynamoDB, SageMaker và IAM PassRole theo Region, resource ARN và tag khi service hỗ trợ.
4. Đặt tên `<WORKSHOP_OPERATOR_POLICY>` và thêm description/owner.
5. Chọn **Create policy**.
6. Quay lại **User groups** → `movie-recommendation-workshop-operators` → **Permissions** → **Add permissions** → **Attach policies**.
7. Chọn `<WORKSHOP_OPERATOR_POLICY>` → **Add permissions**.

Policy dành cho operator cần được security owner xây dựng từ các nhóm quyền sau:

| Nhóm thao tác | Quyền cần cấp theo phạm vi |
|---|---|
| S3 setup | Tạo bucket, cấu hình ownership, Block Public Access, versioning, encryption, tag và thao tác object trên đúng bucket/prefix |
| DynamoDB setup | Tạo/mô tả năm bảng, cấu hình TTL, PITR, encryption/tag và đọc/ghi dữ liệu workshop |
| SageMaker setup | Tạo/mô tả Processing Job, Model, EndpointConfig, Endpoint; invoke endpoint và xem log/metric |
| EC2 setup | Launch/describe instance, tạo/gắn security group, quản lý rule, tag và gắn đúng instance profile |
| CloudWatch | Tạo log group, retention, metric filter, alarm và đọc log/metric workshop |
| Budgets | Xem/tạo/sửa đúng budget và cấu hình notification đã được phê duyệt |
| PassRole | `iam:PassRole` chỉ cho SageMaker execution role và EC2 role được chỉ định |

{{% notice warning %}}
Không gắn `AdministratorAccess` hoặc `IAMFullAccess` để bỏ qua lỗi permission. Nếu dùng AWS managed FullAccess policy trong account lab cô lập, phải ghi rõ đây là ngoại lệ tạm thời, đặt thời hạn gỡ và không tái sử dụng cho production.
{{% /notice %}}

<!-- IMAGE-5.5-IAM-02: Customer managed policy đã được gắn vào operator group. -->

## 4. Tạo service role cho SageMaker và EC2

### 4.1. SageMaker execution role

1. Vào **IAM** → **Roles** → **Create role**.
2. Chọn **AWS service** → **SageMaker** làm trusted service.
3. Gắn customer managed policy cho phép đọc/ghi đúng S3 input/output prefix, kéo đúng ECR image và ghi CloudWatch Logs.
4. Nếu Processing Job chạy trong VPC, bổ sung các quyền quản lý network interface theo tài liệu SageMaker.
5. Đặt tên `movie-rec-sagemaker-execution-role` và tạo role.
6. Kiểm tra trust relationship chỉ cho `sagemaker.amazonaws.com` assume role.

### 4.2. EC2 instance role

1. Vào **IAM** → **Roles** → **Create role**.
2. Chọn **AWS service** → **EC2**.
3. Gắn backend runtime policy giới hạn trên năm bảng DynamoDB, S3 bucket/prefix và một SageMaker Endpoint.
4. Gắn `CloudWatchAgentServerPolicy` hoặc customer managed policy tương đương để gửi metric/log.
5. Nếu cài agent qua Systems Manager, gắn thêm quyền SSM tối thiểu được platform team phê duyệt.
6. Đặt tên `movie-rec-ec2-application-role`, tạo role và chọn role này trong IAM instance profile khi launch EC2.

<!-- IMAGE-5.5-IAM-03: Trust relationship và permission policies của SageMaker execution role. -->

<!-- IMAGE-5.5-IAM-04: EC2 role/instance profile với backend runtime và CloudWatch Agent permissions. -->

## 5. Ma trận principal và quyền

| Principal | Quyền cần thiết | Phạm vi tài nguyên |
|---|---|---|
| Backend EC2 role | `sts:GetCallerIdentity` | Identity hiện tại |
| Backend EC2 role | DynamoDB describe/get/batch-get/put/query/scan theo code path | ARN của năm bảng và index liên quan |
| Backend EC2 role | S3 list cho startup validation | Một bucket, giới hạn prefix khi có thể |
| Backend EC2 role | `sagemaker:DescribeEndpoint`, `sagemaker:InvokeEndpoint` | Một endpoint |
| S3/ML operator | S3 list/get/put | Data, model và report prefix |
| Processing submitter | Tạo/mô tả Processing Job, `iam:PassRole` | Job namespace và một execution role |
| SageMaker execution role | S3 list/get/put, CloudWatch Logs | Prefix và log group cụ thể |
| GitHub deployer | SSH credential trong GitHub Secrets | Một EC2 host |

{{% notice warning %}}
Không sử dụng `Action: "*"` hoặc `Resource: "*"` chỉ để làm cho hệ thống chạy. Policy production phải được security owner review.
{{% /notice %}}

## 6. Nhận xét về least privilege

- Backend HTTP runtime không cần `DeleteItem` vì hiện không có delete API.
- Quyền S3 upload/download nên thuộc tooling role riêng thay vì mở rộng web runtime role.
- `Users` hiện cần scan cho case-insensitive login/uniqueness vì không có identity GSI.
- Principal triển khai SageMaker và SageMaker execution role là hai principal khác nhau.
- `iam:PassRole` chỉ được phép đối với đúng execution role.

## 7. Bảo mật ứng dụng

- Password sử dụng PBKDF2-HMAC-SHA256 với random salt và số vòng lặp được cấu hình.
- JWT HS256 kiểm tra signature, issuer, audience và expiry.
- Protected action của guest trả `401`.
- User chưa hoàn thành onboarding nhận `403` khi gọi recommendation.
- Frontend lưu access token trong `localStorage`, vì vậy cần kiểm soát XSS.
- Logout hiện không revoke JWT ở server; endpoint chỉ trả `204`.

## 8. Positive test

Với backend role đã được gắn đúng:

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

Tiêu chí đạt: cả ba thao tác được phê duyệt đều thành công.

{{% notice note %}}
`sts:GetCallerIdentity` có hành vi đặc biệt và không đủ để chứng minh principal có quyền truy cập DynamoDB, S3 hoặc SageMaker.
{{% /notice %}}

<!-- IMAGE-5.5-01: IAM role với policy giới hạn tài nguyên, đã che ARN/account ID. -->

## 9. Negative test an toàn

Security owner phải cung cấp một resource ngoài phạm vi được phê duyệt dành riêng cho kiểm thử:

```bash
aws dynamodb describe-table \
  --table-name "<APPROVED_OUT_OF_SCOPE_TEST_TABLE>" \
  --region "<AWS_REGION>"
```

Kết quả mong đợi: `AccessDeniedException`.

`ResourceNotFoundException` không chứng minh least privilege vì resource có thể không tồn tại.

<!-- IMAGE-5.5-02: AccessDenied khi truy cập resource kiểm thử ngoài phạm vi. -->


## 10. Cấu hình Amazon CloudWatch

### 10.1. Tạo log groups và retention

1. Mở **CloudWatch** → **Logs** → **Log groups** → **Create log group**.
2. Tạo các log group theo môi trường, ví dụ:
   - `/movie-rec/<ENVIRONMENT>/system`
   - `/movie-rec/<ENVIRONMENT>/backend`
   - `/movie-rec/<ENVIRONMENT>/frontend`
3. Chọn KMS key nếu chính sách yêu cầu mã hóa bằng customer managed key.
4. Sau khi tạo, chọn log group → **Actions** → **Edit retention setting**.
5. Đặt retention phù hợp, ví dụ 14 hoặc 30 ngày cho workshop; không để `Never expire` nếu không có yêu cầu lưu giữ.

SageMaker tự tạo log group/stream cho Processing Job và Endpoint khi execution role có quyền. Không đổi tên các log group do service quản lý; dùng tag, dashboard và alarm để tập trung quan sát.

![Các CloudWatch log group của SageMaker Endpoint và Processing Jobs](/images/5-Workshop/5.5-IAM-security/cloudwatch-sagemaker-log-groups.png)


### 10.2. Cài CloudWatch Agent trên EC2

1. Xác nhận EC2 instance profile có `CloudWatchAgentServerPolicy` hoặc policy tối thiểu tương đương.
2. Mở **CloudWatch** → **Getting Started** → quy trình **Install and configure CloudWatch agent**.
3. Chọn EC2 instance theo ID hoặc tag `Project=movie-recommendation`.
4. Chọn workload/configuration phù hợp và thu thập ít nhất memory, disk usage cùng các system metrics cần thiết.
5. Khai báo file log thực tế của ứng dụng và hệ điều hành. Với Docker stdout/stderr, cấu hình logging driver hoặc cơ chế chuyển log trước khi khai báo log group đích.
6. Ánh xạ log tới đúng log group và stream name có chứa instance ID/container name.
7. Apply configuration, chờ agent ở trạng thái hoạt động và xác nhận log stream có event mới.

Nếu tùy chọn cài agent bằng Console không có trong Region, dùng AWS Systems Manager hoặc cài thủ công theo tài liệu CloudWatch Agent.

<!-- IMAGE-5.5-CLOUDWATCH-01: CloudWatch Agent configuration và log groups có log stream từ EC2/backend. -->

### 10.3. Tạo alarms

1. Vào **CloudWatch** → **Alarms** → **All alarms** → **Create alarm**.
2. Chọn **Select metric**, chọn service/resource và metric cần giám sát.
3. Chọn statistic, period, threshold và số datapoint cần vi phạm.
4. Tại **Notification**, chọn hoặc tạo SNS topic và nhập email nhận cảnh báo.
5. Xác nhận subscription nếu dùng SNS email.
6. Đặt alarm name chứa environment và resource, review rồi chọn **Create alarm**.

| Resource | Metric gợi ý | Điều kiện workshop tham khảo |
|---|---|---|
| EC2 | `CPUUtilization` | Lớn hơn 80% trong nhiều datapoint liên tiếp |
| EC2 | `StatusCheckFailed` | Lớn hơn hoặc bằng 1 |
| DynamoDB | `ThrottledRequests` và `SystemErrors` | Lớn hơn hoặc bằng 1 trong khoảng đánh giá |
| SageMaker Endpoint | `Invocation5XXErrors` | Lớn hơn hoặc bằng 1 |
| SageMaker Endpoint | `ModelLatency` | Theo dõi percentile và đặt threshold sau khi có baseline |

Các threshold trên chỉ là điểm bắt đầu cho workshop, không phải production SLA.

<!-- IMAGE-5.5-CLOUDWATCH-02: CloudWatch alarm ở trạng thái OK và SNS notification đã cấu hình. -->

## 11. Cấu hình AWS Budgets

### 11.1. Cho phép IAM truy cập Billing

1. Đăng nhập root user cho thao tác một lần này.
2. Mở **Billing and Cost Management** → phần account/billing access settings.
3. Tại **IAM User and Role Access to Billing Information**, chọn **Edit** → bật **Activate IAM Access** → lưu thay đổi.
4. Đăng xuất root ngay sau khi hoàn tất.
5. Gắn policy Budgets/Billing tối thiểu cho operator role hoặc group. Bật IAM access không tự cấp permission.

### 11.2. Tạo monthly cost budget

1. Mở **Billing and Cost Management** → **Budgets** → **Create budget**.
2. Chọn **Customize (advanced)** → **Cost budget** → **Next**.
3. Nhập budget name, ví dụ `movie-recommendation-workshop-monthly`.
4. Chọn period **Monthly**, recurring budget và nhập `<MONTHLY_BUDGET_USD>`.
5. Tại scope/filter, giới hạn theo AWS service, account hoặc cost allocation tag `Project=movie-recommendation` khi tag đã được kích hoạt cho billing.
6. Tạo các alert tham khảo:
   - Actual cost đạt 50%.
   - Actual cost đạt 80%.
   - Forecasted cost đạt 100%.
7. Thêm tối đa các email cần nhận thông báo; có thể thêm SNS topic nếu tổ chức sử dụng kênh cảnh báo tập trung.
8. Với workshop đầu tiên, không bật automatic budget action làm dừng/xóa tài nguyên khi chưa có runbook và phê duyệt.
9. Review → **Create budget** và kiểm tra trạng thái notification.

![AWS Budget theo dõi chi phí của workshop](/images/5-Workshop/5.5-IAM-security/aws-budget-overview.png)

*Billing and Cost Management xác nhận budget `My-200$-budget` có hạn mức 200 USD, threshold ở trạng thái `OK` và health status `Healthy`..*

<!-- IMAGE-5.5-BUDGETS-01: Monthly cost budget với amount, scope và ba alert thresholds. -->
