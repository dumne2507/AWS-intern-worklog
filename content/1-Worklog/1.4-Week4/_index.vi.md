---
title: "Worklog Tuần 4"
date: 2026-10-05
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

## Mục tiêu tuần 4: Phân quyền & IAM

- **IAM Identity Center:** Cấu hình AWS Single Sign-On (SSO) để quản lý đăng nhập tập trung cho toàn bộ tài khoản AWS trong Organization.
- **IAM Policies nâng cao:** Viết custom JSON policies và áp dụng Condition keys để giới hạn truy cập theo địa chỉ IP và khung giờ cụ thể.
- **Permission Boundaries:** Thiết lập ranh giới quyền hạn để kiểm soát tối đa quyền hiệu lực của IAM user và role.
- **IAM Roles cho EC2:** Cấp IAM Role cho EC2 instance để máy chủ tự động truy cập S3/DynamoDB mà không cần lưu trữ thông tin xác thực dài hạn.
- **AWS Secrets Manager:** Lưu trữ an toàn và xoay vòng tự động mật khẩu database, loại bỏ hoàn toàn thông tin xác thực cứng trong code.
- **Lab Tổng hợp 4:** Triển khai kiến trúc EC2 không có thông tin xác thực cứng, gọi API lấy mật khẩu DB động từ Secrets Manager.

### Các công việc cần triển khai trong tuần này

| Thứ | Chủ đề | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **2** | **IAM Identity Center** | Cấu hình AWS SSO để quản lý đăng nhập tập trung | 05/10/2026 | 05/10/2026 | [IAM Identity Center](https://docs.aws.amazon.com/singlesignon/) |
| **3** | **IAM Policies nâng cao** | Viết custom JSON policies, dùng Condition keys giới hạn truy cập theo IP/thời gian | 06/10/2026 | 06/10/2026 | [IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) |
| **4** | **Permission Boundaries** | Thiết lập ranh giới quyền hạn kiểm soát quyền tối đa của user/role | 07/10/2026 | 07/10/2026 | [Permissions Boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) |
| **5** | **IAM Roles cho EC2** | Cấp Role cho EC2 tự động truy cập S3/DynamoDB không cần lưu khóa | 08/10/2026 | 08/10/2026 | [IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) |
| **6** | **AWS Secrets Manager** | Lưu trữ, xoay vòng bảo mật mật khẩu database an toàn | 09/10/2026 | 09/10/2026 | [Secrets Manager](https://docs.aws.amazon.com/secretsmanager/) |
| **7** | **Lab Tổng hợp 4** | Triển khai EC2 không giữ thông tin xác thực cứng, gọi API lấy mật khẩu DB từ Secrets Manager | 10/10/2026 | 10/10/2026 | [AWS Security Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) |

### Kết quả đạt được tuần 4

- Bật AWS IAM Identity Center tại `us-east-1` thành công, tạo Permission Set `DevReadOnly` và xác minh luồng đăng nhập SSO qua URL cổng truy cập AWS.
- Viết custom IAM policies với các Condition key `aws:SourceIp`, `aws:RequestedRegion`, `aws:CurrentTime`; kiểm chứng tất cả kịch bản qua IAM Policy Simulator đều trả đúng kết quả.
- Cấu hình Permission Boundary `DeveloperBoundary`, xác nhận quyền hiệu lực chỉ là giao của identity policy và boundary — kể cả `AdministratorAccess` cũng bị giới hạn.
- Tạo IAM Role `EC2-S3-DynamoDB-Role` gắn vào EC2, xác minh truy cập S3 và DynamoDB không cần thông tin xác thực tĩnh qua `aws sts get-caller-identity`.
- Lưu thông tin xác thực RDS `prod/myapp/db-password` vào Secrets Manager với mã hóa KMS, bật xoay vòng tự động 30 ngày qua Lambda.
- Hoàn thành xuất sắc Lab Tổng hợp 4: kiến trúc zero-credential hoàn chỉnh — IMDS → IAM Role → Secrets Manager → KMS giải mã → mật khẩu trong bộ nhớ — không có thông tin xác thực nào ghi ra ổ đĩa.

---

#### Bài tập 1: IAM Identity Center — Quản lý đăng nhập SSO tập trung

AWS IAM Identity Center (trước đây là AWS SSO) cung cấp một điểm đăng nhập duy nhất cho tất cả tài khoản AWS và ứng dụng trong một AWS Organization. Tôi bắt đầu bằng cách bật IAM Identity Center tại vùng **us-east-1** và liên kết với tài khoản root của Organization. Dịch vụ tạo ra một **Identity Store** riêng biệt, đóng vai trò là nguồn thông tin danh tính chính thức.

Sau khi bật dịch vụ, tôi tạo một **Permission Set** tên `DevReadOnly` bao gồm policy quản lý `ReadOnlyAccess`. Permission Set định nghĩa những gì người dùng IAM Identity Center có thể làm sau khi đăng nhập vào một tài khoản cụ thể. Tiếp theo, tôi tạo một user, gán Permission Set cho user đó trên tài khoản development, rồi kiểm tra luồng đăng nhập qua URL cổng truy cập AWS được tạo tự động.

URL cổng SSO theo định dạng `https://<subdomain>.awsapps.com/start`. Từ cổng này, người dùng có thể chuyển đổi giữa các tài khoản AWS chỉ bằng một cú nhấp chuột, và thông tin xác thực phiên tạm thời được tạo tự động trong nền — không cần access key dài hạn.

> **Screenshot:** ![Bảng điều khiển IAM Identity Center SSO](/images/1-Worklog/week-04/01-iam-identity-center-sso.png)

---

#### Bài tập 2: IAM Policies nâng cao — Condition Keys

Policy IAM thông thường cho phép hoặc từ chối hành động trên tài nguyên, nhưng **Condition keys** giúp policy trở nên linh hoạt hơn. Tôi tạo một custom policy cho phép `s3:GetObject` chỉ khi IP của người gọi thuộc dải CIDR nội bộ `203.0.113.0/24` thông qua condition key `aws:SourceIp`.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-intern-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

Tôi cũng thử nghiệm `aws:RequestedRegion` để chỉ cho phép gọi API trong vùng `ap-southeast-1`, và `aws:CurrentTime` để giới hạn truy cập console trong giờ làm việc (08:00–18:00 UTC+7). Các Condition key này được IAM engine kiểm tra theo thời gian thực trước khi bất kỳ hành động nào được cho phép.

> **Screenshot:** ![Trình soạn thảo JSON Policy với Condition Keys](/images/1-Worklog/week-04/02-iam-policy-json-editor.png)

Để xác minh policy, tôi sử dụng công cụ **IAM Policy Simulator** trong AWS Console. Công cụ này cho phép chọn user hoặc role, chỉ định action và resource, rồi xem chính xác câu lệnh policy nào khớp — Allow hay Deny — mà không cần thực hiện lời gọi API thực tế.

> **Screenshot:** ![Chi tiết cấu hình Condition Key](/images/1-Worklog/week-04/03-iam-condition-keys.png)

Sau khi mô phỏng nhiều kịch bản (đúng IP, sai IP, trong giờ làm việc, ngoài giờ làm việc), tất cả kết quả đều đúng như kỳ vọng: yêu cầu từ CIDR không được phép hoặc ngoài khung giờ trả về `ImplicitDeny`.

> **Screenshot:** ![Kết quả kiểm tra Policy Simulator](/images/1-Worklog/week-04/04-iam-policy-test-result.png)

---

#### Bài tập 3: Permission Boundaries — Giới hạn quyền tối đa

**Permission Boundary** là tính năng IAM nâng cao giúp thiết lập giới hạn quyền tối đa mà một identity-based policy có thể cấp cho IAM entity (user hoặc role). Dù policy của entity có cho phép một hành động, nếu permission boundary không cho phép, hành động đó vẫn bị từ chối. Tính năng này đặc biệt hữu ích cho quản trị ủy quyền — cho phép admin cấp dưới tạo IAM role nhưng ngăn leo thang đặc quyền.

Tôi tạo một boundary policy tên `DeveloperBoundary` chỉ cho phép `s3:*`, `dynamodb:*`, và `cloudwatch:GetMetric*`. Sau đó tạo IAM role `AppRole-Dev` và gắn boundary này qua AWS CLI:

```bash
aws iam create-role \
  --role-name AppRole-Dev \
  --assume-role-policy-document file://trust-policy.json \
  --permissions-boundary arn:aws:iam::123456789012:policy/DeveloperBoundary
```

> **Screenshot:** ![Thiết lập Permission Boundary trong Console](/images/1-Worklog/week-04/05-permission-boundary-setup.png)

Sau khi gắn boundary, tôi cũng gắn `AdministratorAccess` làm identity-based policy cho cùng role (cố ý đặt quyền quá rộng). Thử gọi `ec2:DescribeInstances` qua role này trả về `AccessDenied` vì EC2 không có trong boundary — minh chứng rằng boundary giới hạn quyền bất kể identity policy cho phép gì.

> **Screenshot:** ![Chi tiết policy Permission Boundary](/images/1-Worklog/week-04/06-permission-boundary-policy.png)

Tôi chỉnh sửa boundary để thêm `ec2:DescribeInstances` rồi kiểm tra lại — lần này lời gọi thành công, xác nhận boundary chính là yếu tố kiểm soát quyền truy cập.

> **Screenshot:** ![Kết quả kiểm tra Deny với Boundary](/images/1-Worklog/week-04/07-permission-boundary-deny-test.png)

Cuối cùng, tôi chạy IAM Policy Simulator có tính đến boundary và xác nhận quyền hiệu lực chỉ là giao giữa identity policy và boundary policy.

> **Screenshot:** ![Kết quả cuối cùng Permission Boundary](/images/1-Worklog/week-04/08-permission-boundary-result.png)

---

#### Bài tập 4: IAM Roles cho EC2 — Kiến trúc không lưu khóa cứng

Lưu thông tin xác thực AWS (`aws_access_key_id`, `aws_secret_access_key`) trực tiếp vào code hoặc biến môi trường của EC2 là lỗ hổng bảo mật nghiêm trọng. Giải pháp đúng đắn là gắn **IAM Role** cho EC2 instance. Dịch vụ Instance Metadata Service (IMDS) sẽ tự động cung cấp thông tin xác thực tạm thời, xoay vòng liên tục cho mọi tiến trình trên máy — không cần access key dài hạn.

Tôi tạo IAM role tên `EC2-S3-DynamoDB-Role` với trust policy cho EC2 và gắn hai managed policy: `AmazonS3ReadOnlyAccess` và `AmazonDynamoDBReadOnlyAccess`. Sau đó liên kết role với EC2 instance đang chạy qua AWS CLI:

```bash
aws ec2 associate-iam-instance-profile \
  --instance-id i-0abcdef1234567890 \
  --iam-instance-profile Name=EC2-S3-DynamoDB-Role
```

> **Screenshot:** ![IAM Role được gắn vào EC2 Instance](/images/1-Worklog/week-04/09-iam-role-ec2-attach.png)

Từ bên trong EC2, AWS CLI tự động sử dụng thông tin xác thực từ role mà không cần `aws configure`:

```bash
# Bên trong EC2 — không cần aws configure
aws s3 ls s3://my-intern-bucket
aws dynamodb list-tables --region ap-southeast-1

# Xác minh danh tính
aws sts get-caller-identity
```

Cả hai lệnh đều thành công và `aws sts get-caller-identity` xác nhận danh tính người gọi là IAM Role, không phải IAM user. Thông tin xác thực tạm thời được metadata service tự động làm mới trước khi hết hạn.

---

#### Bài tập 5: AWS Secrets Manager — Lưu trữ & Xoay vòng thông tin xác thực an toàn

Lưu mật khẩu database, API key dưới dạng plaintext (biến môi trường, file cấu hình, mã nguồn) là rủi ro bảo mật lớn. **AWS Secrets Manager** giải quyết vấn đề này bằng cách lưu trữ secret dưới dạng mã hóa (dùng AWS KMS) và cung cấp API lấy giá trị lúc runtime, không bao giờ lưu ra đĩa.

Tôi tạo secret mới loại **RDS credentials** chứa username và password của MySQL RDS instance. Secrets Manager mã hóa giá trị và lưu dưới tên `prod/myapp/db-password`. Từ EC2 có role phù hợp, ứng dụng lấy secret khi khởi động:

```python
import boto3, json

client = boto3.client('secretsmanager', region_name='ap-southeast-1')
response = client.get_secret_value(SecretId='prod/myapp/db-password')
secret = json.loads(response['SecretString'])
DB_PASSWORD = secret['password']
```

> **Screenshot:** ![Tạo Secret trong Secrets Manager](/images/1-Worklog/week-04/10-secrets-manager-create.png)

Tiếp theo tôi bật **xoay vòng tự động** với lịch 30 ngày. Secrets Manager sử dụng Lambda function (tự tạo cho loại RDS) để tạo mật khẩu mới, cập nhật trong RDS instance và lưu giá trị mới — tất cả không gây downtime. Ứng dụng chỉ cần gọi `get_secret_value` mỗi lần cần mật khẩu, không bao giờ cache hay lưu giá trị.

> **Screenshot:** ![Cấu hình xoay vòng trong Secrets Manager](/images/1-Worklog/week-04/11-secrets-manager-rotate.png)

---

#### Bài tập 6: Lab Tổng hợp 4 — Kiến trúc EC2 không có thông tin xác thực cứng

Lab cuối tuần tích hợp tất cả kiến thức trong tuần vào một kiến trúc bảo mật cấp production: EC2 instance không lưu bất kỳ thông tin xác thực cứng nào, xác thực qua IAM Role và lấy mật khẩu database động từ Secrets Manager lúc runtime.

Luồng kiến trúc hoạt động như sau:
1. EC2 khởi động → IMDS cung cấp thông tin xác thực tạm thời cho `EC2-AppRole`
2. Ứng dụng gọi `secretsmanager:GetSecretValue` lấy `prod/myapp/db-password`
3. Secrets Manager kiểm tra IAM policy của role → trả về secret đã mã hóa
4. KMS giải mã giá trị → ứng dụng nhận mật khẩu plaintext trong bộ nhớ
5. Ứng dụng kết nối RDS bằng mật khẩu lấy được — không bao giờ ghi ra ổ đĩa

Toàn bộ thành phần được triển khai qua AWS CLI và xác minh end-to-end. Kết quả `aws sts get-caller-identity` xác nhận không có thông tin xác thực IAM user nào tồn tại trên instance ở bất kỳ thời điểm nào.

> **Screenshot:** ![Kiến trúc cuối Lab Tổng hợp 4](/images/1-Worklog/week-04/12-lab4-final-architecture.png)

---

#### Khó khăn gặp phải & Cách giải quyết

| Vấn đề | Nguyên nhân | Cách giải quyết |
| :--- | :--- | :--- |
| SSO bị vòng lặp đăng nhập sau khi bật Identity Center | DNS chưa propagate cho subdomain SSO | Chờ 5 phút và xóa cache trình duyệt trước khi thử lại |
| `AccessDenied` khi kiểm tra Permission Boundary | `ec2:DescribeInstances` chưa có trong boundary policy | Bổ sung action EC2 cụ thể vào policy `DeveloperBoundary` |
| Credentials của `aws configure` ghi đè role EC2 | File `~/.aws/credentials` còn lưu access key tĩnh | Xóa file `~/.aws/credentials`; thông tin xác thực lấy hoàn toàn qua IMDS |
| `ResourceNotFoundException` trong Secrets Manager | Tên secret phân biệt hoa thường, gõ sai trong code | Sửa tên từ `prod/myapp/DB-password` thành `prod/myapp/db-password` |

### Kế hoạch tuần 5: Giám sát & Bảo mật

- Nghiên cứu **Amazon CloudWatch**: metrics, log groups, alarm và dashboard cho EC2, RDS và Lambda.
- Bật **AWS CloudTrail** để kiểm toán tất cả lời gọi API trong tài khoản và lưu trữ log vào S3.
- Cấu hình **Amazon GuardDuty** phát hiện mối đe dọa tự động bằng phân tích bất thường ML.
- Thiết lập quy tắc **AWS Config** để liên tục kiểm tra cấu hình tài nguyên với baseline tuân thủ.
- Khám phá **AWS Security Hub** để tổng hợp phát hiện từ GuardDuty, Config và Inspector vào một dashboard duy nhất.
- Thực hành luồng phản hồi sự cố: GuardDuty phát hiện → EventBridge rule → SNS cảnh báo → Lambda tự động khắc phục.
