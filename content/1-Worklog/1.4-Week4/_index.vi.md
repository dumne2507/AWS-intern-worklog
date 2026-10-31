---
title: "Worklog Tuần 4"
date: 2026-10-05
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

## Mục tiêu tuần 4: Phân quyền & IAM

- **IAM Identity Center:** Cấu hình AWS Single Sign-On (SSO) để quản lý đăng nhập tập trung cho toàn bộ tài khoản AWS trong Organization, kích hoạt xác thực đa yếu tố MFA.
- **IAM Policies nâng cao:** Viết custom JSON policy và áp dụng Condition keys (`aws:SourceIp`) để giới hạn dải IP được phép truy cập tài nguyên S3.
- **Permission Boundaries:** Thiết lập ranh giới quyền hạn tối đa cho IAM User/Role, kiểm soát chặt chẽ nhằm ngăn chặn hành vi leo thang đặc quyền.
- **IAM Roles cho EC2:** Gắn IAM Role trực tiếp cho máy chủ EC2 để tự động truy cập dịch vụ AWS (S3, DynamoDB) mà không cần lưu trữ Access Key/Secret Key tĩnh.
- **AWS KMS & Mã hóa lưu trữ:** Khởi tạo Customer Managed Key (KMS Key) và áp dụng mã hóa dữ liệu lưu trữ cho ổ đĩa EBS Volume và kho lưu trữ S3 Bucket.
- **Lab Tổng hợp 4:** Triển khai kiến trúc máy chủ EC2 không lưu trữ thông tin xác thực, xác thực qua IAM Role và gọi API lấy mật khẩu an toàn từ AWS Secrets Manager.

### Các công việc cần triển khai trong tuần này

| Thứ | Chủ đề | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **2** | **IAM Identity Center** | Cấu hình AWS SSO, thiết lập cổng truy cập Access Portal và kích hoạt bảo mật MFA | 05/10/2026 | 05/10/2026 | [IAM Identity Center](https://docs.aws.amazon.com/singlesignon/) |
| **3** | **IAM Policies nâng cao** | Viết custom JSON policy với Condition key `aws:SourceIp` giới hạn IP truy cập S3 Bucket | 06/10/2026 | 06/10/2026 | [IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) |
| **4** | **Permission Boundaries** | Thiết lập ranh giới quyền hạn kiểm soát quyền tối đa, chặn truy cập ngoài phạm vi cho phép | 07/10/2026 | 07/10/2026 | [Permissions Boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) |
| **5** | **IAM Roles cho EC2** | Tạo và gắn IAM Role cho EC2, xác minh truy cập S3/DynamoDB qua IMDS không cần lưu khóa | 08/10/2026 | 08/10/2026 | [IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) |
| **6** | **AWS KMS & Encryption** | Tạo Customer Managed Key (`Key-KMS-LabT6`), cấu hình mã hóa EBS Volume và S3 Bucket | 09/10/2026 | 09/10/2026 | [AWS KMS](https://docs.aws.amazon.com/kms/) |
| **7** | **Lab Tổng hợp 4** | Triển khai EC2 không giữ thông tin xác thực cứng, gọi API lấy secret từ Secrets Manager | 10/10/2026 | 10/10/2026 | [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/) |

### Kết quả đạt được tuần 4

- Cấu hình thành công AWS IAM Identity Center (SSO) với cổng truy cập `ssoins-72232edfad9c07c3.portal.us-east-1.app.aws`, kích hoạt thiết bị MFA ảo `Admin-Users's MFA 1` bảo vệ tài khoản quản trị `Dũng Nguyễn Mạnh`.
- Xây dựng IAM Policy nâng cao với Condition key `aws:SourceIp` áp dụng cho người dùng `LabT3-Test`, kiểm chứng thành công cơ chế chặn truy cập trái phép (`Insufficient permissions`) từ IP lạ và cho phép truy cập đầy đủ file `LabT3.html` từ IP hợp lệ.
- Áp dụng thành công Permission Boundary cho người dùng `LabT3-Test`: dù có quyền IAM rộng, người dùng vẫn bị hệ thống từ chối triệt để khi truy cập EC2 (`Access denied`) và VPC (`ec2:DescribeAccountAttributes`, `ec2:DescribeVpcs`) do nằm ngoài ranh giới cho phép.
- Triển khai máy chủ EC2 `Lab-T5` (`i-04c89eaec696a2cef`) gắn IAM Role riêng biệt, truy vấn thành công S3 bucket `labt3-test-ip-public` và DynamoDB `list-tables` mà không cần cấu hình file `~/.aws/credentials`.
- Tạo Customer Managed Key `Key-KMS-LabT6` (`1c0663df-33b2-437c-a98f-cc38935054f8`), mã hóa thành công ổ đĩa EBS Volume và thiết lập Default Encryption chuẩn SSE-KMS kèm Bucket Key trên S3 Bucket `labt6-s3-kms`.
- Hoàn thiện xuất sắc Lab Tổng hợp 4: triển khai EC2 instance `EC2-Secret-Role-T7` (`i-032c941e009326a97`), gọi lệnh `aws secretsmanager get-secret-value` lấy secret `secret-labt7` an toàn tuyệt đối từ bộ nhớ, không lưu trữ thông tin xác thực cứng trên máy chủ.

---

#### Bài tập 1: IAM Identity Center — Cấu hình AWS SSO & Kích hoạt MFA Tập trung

Khởi tạo và cấu hình dịch vụ **AWS IAM Identity Center** (trước đây là AWS Single Sign-On) tại Region `us-east-1` nhằm quản lý danh tính tập trung và phân quyền truy cập đa tài khoản:
- **Cổng đăng nhập (Portal URL):** `https://ssoins-72232edfad9c07c3.portal.us-east-1.app.aws/#/security`
- **Tài khoản người dùng:** `Dũng Nguyễn Mạnh` (Username hiển thị: `Dũng`)
- **Thiết bị MFA đã đăng ký:** `Admin-Users's MFA 1`
- **Loại thiết bị:** `Authenticator app` (Xác thực 2 bước trên ứng dụng di động)
- **Thời gian khởi tạo MFA:** `Fri Oct 23 2026 11:03:00 GMT+0700 (Indochina Time)`
- **Phiên đăng nhập hiện tại (Active Session):** `Desktop - Edge 155.0, Windows 10`
- **Địa chỉ IP máy truy cập:** `115.78.22...` | **Thời gian phiên còn lại:** `7 hours 59 minutes`

Cổng AWS Access Portal cho phép chuyển đổi giữa các tài khoản AWS trong Organization chỉ với một lần đăng nhập (SSO), hệ thống tự động cấp phát token phiên ngắn hạn mà không cần lưu trữ Access Key dài hạn trên máy tính cá nhân.

> **Screenshot:** ![Bảng điều khiển AWS Access Portal và thiết lập MFA](/images/1-Worklog/week-04/01-iam-identity-center-sso.png)

---

#### Bài tập 2: IAM Policies nâng cao — Custom Policy & Condition Key Giới hạn IP

Khởi tạo người dùng IAM mới mang tên **`LabT3-Test`** thuộc tài khoản `468502414950` (`Manhdung2507`) để thực hành phân quyền với **Condition Keys**.

> **Screenshot:** ![Danh sách người dùng IAM quản lý trong hệ thống](/images/1-Worklog/week-04/02-iam-policy-json-editor.png)

Thiết lập một custom policy JSON gán cho người dùng `LabT3-Test`, sử dụng Condition key `aws:SourceIp` để chỉ cho phép các thao tác `s3:ListBucket` và `s3:GetObject` tới S3 Bucket **`labt3-test-ip-public`** khi yêu cầu xuất phát từ đúng địa chỉ IP công cộng của quản trị viên:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3AccessFromCorporateIPOnly",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::labt3-test-ip-public",
        "arn:aws:s3:::labt3-test-ip-public/*"
      ],
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "115.78.22.0/24"
        }
      }
    }
  ]
}
```

Tiến hành kiểm thử truy cập theo 2 kịch bản:
- **Kịch bản 1 (IP không hợp lệ / IP lạ):** Khi đăng nhập tài khoản `LabT3-Test` từ dải IP ngoài danh sách cho phép và truy cập bucket `labt3-test-ip-public`, AWS S3 lập tức chặn lại với thông báo lỗi màu đỏ: *"Insufficient permissions to list objects. After you or your AWS administrator has updated your permissions to allow the s3:ListBucket action, refresh the page."*

> **Screenshot:** ![Truy cập S3 bị từ chối do IP không nằm trong Condition Key](/images/1-Worklog/week-04/03-iam-condition-keys.png)

- **Kịch bản 2 (IP hợp lệ):** Khi truy cập từ đúng địa chỉ IP đã khai báo trong Condition block, người dùng `LabT3-Test` duyệt bucket thành công và thấy tệp tin **`LabT3.html`** (Type: `html`, Size: `0 B`, Storage class: `Standard`).

> **Screenshot:** ![Truy cập tệp tin S3 thành công khi kiểm tra đúng IP](/images/1-Worklog/week-04/04-iam-policy-test-result.png)

---

#### Bài tập 3: Permission Boundaries — Thiết lập Ranh giới Quyền hạn Tối đa

Cấu hình tính năng **Permission Boundary** cho người dùng **`LabT3-Test`** (`arn:aws:iam::468502414950:user/LabT3-Test`). Ranh giới chỉ cho phép tương tác với dịch vụ Amazon S3, cụ thể là bucket **`labt3-test-ip-public`** tại vùng `us-east-1`.

> **Screenshot:** ![S3 Bucket labt3-test-ip-public nằm trong phạm vi Permission Boundary](/images/1-Worklog/week-04/05-permission-boundary-setup.png)

Tiến hành kiểm chứng cơ chế kiểm soát ranh giới quyền hạn:
- **Kiểm tra trên EC2 Dashboard:** Khi người dùng `LabT3-Test` truy cập bảng điều khiển EC2, toàn bộ tài nguyên hiển thị trạng thái **`Access denied`**:
  - `Dedicated Hosts:` ❌ Access denied
  - `Key pairs:` ❌ Access denied
  - `Security groups:` ❌ Access denied
  - `Auto Scaling Groups:` ❌ Access denied
  - `Elastic IPs:` ❌ Access denied
  - `Load balancers:` ❌ Access denied
  - `Instances (running):` ❌ Access denied
  - `Volumes:` ❌ Access denied
  - `Service health:` ❌ *You don't have permission to health:DescribeEvents.*

> **Screenshot:** ![Bảng điều khiển EC2 bị từ chối truy cập toàn diện](/images/1-Worklog/week-04/06-permission-boundary-policy.png)

- **Kiểm tra trên VPC Dashboard:** Khi truy cập trang quản lý VPC, giao diện xuất hiện banner cảnh báo lỗi màu đỏ từ chối thao tác:
  - Thông báo: *"User: arn:aws:iam::468502414950:user/LabT3-Test is not authorized to perform: ec2:DescribeAccountAttributes because no permissions boundary allows the ec2:DescribeAccountAttributes action"*

> **Screenshot:** ![Lỗi từ chối quyền DescribeAccountAttributes do Permission Boundary](/images/1-Worklog/week-04/07-permission-boundary-deny-test.png)

- **Kiểm tra danh sách Your VPCs:** Hệ thống tiếp tục chặn truy vấn danh sách VPC với cảnh báo:
  - Thông báo: *"User: arn:aws:iam::468502414950:user/LabT3-Test is not authorized to perform: ec2:DescribeVpcs because no permissions boundary allows the ec2:DescribeVpcs action"*

> **Screenshot:** ![Lỗi từ chối quyền DescribeVpcs khi người dùng bị giới hạn quyền](/images/1-Worklog/week-04/08-permission-boundary-result.png)

Kết quả kiểm thử chứng minh: Dù người dùng được gán chính sách quyền hạn rộng đến đâu, nếu hành động không nằm trong **Permission Boundary**, quyền hiệu lực thực tế vẫn là **Deny**, bảo vệ vững chắc tài nguyên quản trị.

---

#### Bài tập 4: IAM Roles cho EC2 — Truy cập Dịch vụ Không Cần Lưu Khóa Cứng

Khởi chạy một máy chủ ảo EC2 chuyên dụng cho bài lab:
- **Tên máy chủ:** **`Lab-T5`**
- **Instance ID:** `i-04c89eaec696a2cef`
- **Hệ điều hành:** Amazon Linux 2023
- **Public IPv4:** `54.87.203.146`
- **Private IPv4:** `172.31.34.206`
- **Availability Zone:** `us-east-1`
- **IAM Role gắn kết:** Cấp quyền đọc Amazon S3 (`s3:ListAllMyBuckets`, `s3:Get*`) và Amazon DynamoDB (`dynamodb:ListTables`).

Kết nối vào instance qua SSH / EC2 Instance Connect và thực hiện kiểm tra quyền truy cập thông qua AWS CLI:

```bash
# Liệt kê danh sách S3 Buckets qua Role
[ec2-user@ip-172-31-34-206 ~]$ aws s3 ls
2026-10-23 06:13:30 labt3-test-ip-public

# Liệt kê danh sách bảng DynamoDB trong region us-east-1
[ec2-user@ip-172-31-34-206 ~]$ aws dynamodb list-tables --region us-east-1
{
    "TableNames": []
}
```

Cả hai lệnh đều thực thi thành công mỹ mãn mà không cần chạy lệnh `aws configure`, không tồn tại tệp tin `~/.aws/credentials`. Dịch vụ Instance Metadata Service (IMDSv2) tự động cung cấp thông tin xác thực tạm thời và định kỳ xoay vòng khóa, đảm bảo an toàn tối đa cho ứng dụng.

> **Screenshot:** ![Máy chủ EC2 thực thi lệnh S3 và DynamoDB thành công qua IAM Role](/images/1-Worklog/week-04/09-iam-role-ec2-attach.png)

---

#### Bài tập 5: AWS KMS & Mã hóa Lưu trữ — Bảo vệ Ổ đĩa EBS & S3 Bucket

Khởi tạo một khóa mã hóa do người dùng quản lý (**Customer Managed Key - KMS Key**) mang bí danh (Alias) **`Key-KMS-LabT6`**:
- **KMS Key ID:** `1c0663df-33b2-437c-a98f-cc38935054f8`
- **KMS Key ARN:** `arn:aws:kms:us-east-1:468502414950:key/1c0663df-33b2-437c-a98f-cc38935054f8`
- **Vùng hoạt động:** `us-east-1` (N. Virginia)

Ứng dụng khóa KMS vào 2 dịch vụ lưu trữ cốt lõi:
- **Mã hóa khối lưu trữ EBS Volume:** Tạo ổ đĩa EBS mới và kích hoạt mã hóa dữ liệu. Bảng điều khiển EC2 Volumes hiển thị trạng thái:
  - **Status check:** `Okay`
  - **Encryption:** `Encrypted`
  - **KMS key ID:** `1c0663df-33b...` (Liên kết trực tiếp tới alias `Key-KMS-LabT6`)

> **Screenshot:** ![EBS Volume được mã hóa an toàn bằng Customer Managed Key](/images/1-Worklog/week-04/10-secrets-manager-create.png)

- **Mã hóa kho lưu trữ đối tượng S3 Bucket:** Tạo S3 Bucket mang tên **`labt6-s3-kms`** và cấu hình **Default encryption**:
  - **Encryption type:** `Server-side encryption with AWS Key Management Service keys (SSE-KMS)`
  - **Encryption key ARN:** `arn:aws:kms:us-east-1:468502414950:key/1c0663df-33b2-437c-a98f-cc38935054f8`
  - **Bucket Key:** `Enabled` (Tự động giảm tải số lượng API gọi đến KMS, tiết kiệm tới 99% chi phí giải mã)
  - **Blocked encryption types:** `Server-Side Encryption with Customer-Provided Keys (SSE-C)`

> **Screenshot:** ![S3 Bucket labt6-s3-kms cấu hình mã hóa mặc định SSE-KMS](/images/1-Worklog/week-04/11-secrets-manager-rotate.png)

---

#### Bài tập 6: Lab Tổng hợp 4 — Kiến trúc EC2 Không Lưu Khóa, Lấy Secret từ AWS Secrets Manager

Xây dựng kiến trúc bảo mật cấp doanh nghiệp: Triển khai máy chủ ứng dụng EC2 hoàn toàn không lưu trữ bất kỳ thông tin mật khẩu hay chuỗi kết nối nào trong tệp cấu hình. Máy chủ được gắn IAM Role có quyền truy xuất dịch vụ **AWS Secrets Manager**:
- **Tên máy chủ EC2:** **`EC2-Secret-Role-T7`**
- **Instance ID:** `i-032c941e009326a97`
- **Hệ điều hành:** Amazon Linux 2023
- **Public IPv4:** `100.54.219.219`
- **Private IPv4:** `172.31.7.232`
- **Tên Secret:** `secret-labt7`
- **Secret ARN:** `arn:aws:secretsmanager:us-east-1:468502414950:secret:secret-labt7-JCzzxK`
- **Version ID:** `dfba26c0-ef0f-4379-8c7a-bd024eca8ef3`
- **Trạng thái Version:** `["AWSCURRENT"]`

Từ terminal của máy chủ `EC2-Secret-Role-T7`, thực hiện lệnh gọi API lấy mật khẩu an toàn lúc chạy:

```bash
# Kết nối máy chủ EC2-Secret-Role-T7 (IP: 172.31.7.232)
Last login: Sat Oct 31 15:19:14 2026 from 18.206.107.29

# Gọi API AWS Secrets Manager lấy giá trị secret runtime
[ec2-user@ip-172-31-7-232 ~]$ aws secretsmanager get-secret-value --secret-id secret-labt7 --region us-east-1
{
    "ARN": "arn:aws:secretsmanager:us-east-1:468502414950:secret:secret-labt7-JCzzxK",
    "Name": "secret-labt7",
    "VersionId": "dfba26c0-ef0f-4379-8c7a-bd024eca8ef3",
    "SecretString": "{\"secretkey123\":\"Secretkey123\"}",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": "2026-10-31T15:09:04.069000+00:00"
}
```

Hệ thống trả về chính xác chuỗi JSON chứa khóa bí mật `{"secretkey123":"Secretkey123"}` ngay trong bộ nhớ RAM của tiến trình. Mô hình này triệt tiêu hoàn toàn nguy cơ rò rỉ khóa khi mã nguồn bị lộ hoặc máy chủ bị xâm nhập trái phép.

> **Screenshot:** ![EC2 lấy secret thành công từ AWS Secrets Manager](/images/1-Worklog/week-04/12-lab4-final-architecture.png)

---

#### Khó khăn gặp phải & Cách giải quyết

| Vấn đề | Nguyên nhân | Cách giải quyết |
| :--- | :--- | :--- |
| Không thể truy cập S3 Bucket dù đã gắn Policy cho User | Condition key `aws:SourceIp` chặn các địa chỉ IP không nằm trong dải cho phép | Kiểm tra IP công cộng thực tế hiện tại của máy và cập nhật dải CIDR chính xác vào Condition block |
| Lỗi `AccessDenied` khi thao tác trên EC2/VPC console | Người dùng bị giới hạn bởi Permission Boundary (chỉ cho phép S3) | Đây là hành vi đúng của Permission Boundary nhằm cô lập quyền hạn; nếu cần thao tác EC2/VPC thì quản trị viên phải cập nhật Boundary Policy |
| Lỗi `ec2:DescribeAccountAttributes` khi vào giao diện VPC | Ranh giới quyền hạn không cấp quyền mô tả thuộc tính tài khoản | Bổ sung action `ec2:DescribeAccountAttributes` vào chính sách ranh giới quyền hạn nếu muốn người dùng xem được giao diện VPC |
| Lệnh gọi Secrets Manager trên EC2 bị lỗi Unauthorized | Instance Profile của EC2 chưa được cấp quyền `secretsmanager:GetSecretValue` | Gắn IAM Policy cho phép action `secretsmanager:GetSecretValue` tới đúng ARN của secret `secret-labt7` vào IAM Role của máy chủ |
| Chi phí gọi KMS API tăng khi có nhiều lượt đọc trên S3 | Mỗi lần đọc object mã hóa SSE-KMS đều phát sinh một request giải mã tới KMS | Kích hoạt tính năng **Bucket Key** trong cấu hình Default Encryption của S3 để giảm tới 99% chi phí gọi KMS |

### Kế hoạch tuần 5: Giám sát & Bảo mật

- Nghiên cứu **Amazon CloudWatch**: metrics, log groups, alarm và dashboard cho EC2, RDS và Lambda.
- Bật **AWS CloudTrail** để kiểm toán tất cả lời gọi API trong tài khoản và lưu trữ log vào S3.
- Cấu hình **Amazon GuardDuty** phát hiện mối đe dọa tự động bằng phân tích bất thường ML.
- Thiết lập quy tắc **AWS Config** để liên tục kiểm tra cấu hình tài nguyên với baseline tuân thủ.
- Khám phá **AWS Security Hub** để tổng hợp phát hiện từ GuardDuty, Config và Inspector vào một dashboard duy nhất.
- Thực hành luồng phản hồi sự cố: GuardDuty phát hiện → EventBridge rule → SNS cảnh báo → Lambda tự động khắc phục.
