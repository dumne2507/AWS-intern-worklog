---
title: "Worklog Tuần 1"
date: 2026-09-14
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

## Mục tiêu tuần 1

- Thiết lập tài khoản AWS và cấu hình bảo mật cơ bản (MFA, AWS CLI).
- Nắm vững IAM: User, Group, Policy và thiết lập cảnh báo chi phí với AWS Budgets.
- Khởi chạy và quản lý máy chủ ảo EC2, kết nối SSH, cấu hình Elastic IP.
- Làm việc với ổ cứng EBS: tạo, gắn kết, định dạng và chụp Snapshot.
- Tạo và quản lý S3 Bucket, cấu hình quyền truy cập public/private.
- Lab tổng hợp: dựng website tĩnh trên S3 và chạy web server trên EC2.

### Các công việc cần triển khai trong tuần này

| Thứ | Chủ đề | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- | --- |
| 2 | AWS & Tài khoản | Khởi tạo AWS, thiết lập bảo mật MFA, tạo IAM User, cấu hình AWS CLI | 14/09/2026 | 14/09/2026 | [AWS Free Tier](https://aws.amazon.com/free/) |
| 3 | IAM & Budgets | Tạo IAM User/Group, phân quyền cơ bản. Cấu hình cảnh báo chi phí (AWS Budgets) | 15/09/2026 | 15/09/2026 | [AWS IAM](https://000002.awsstudygroup.com) |
| 4 | Amazon EC2 | Khởi chạy máy chủ ảo EC2, tạo Key Pair, kết nối SSH, thực hành Elastic IP | 16/09/2026 | 16/09/2026 | [Amazon EC2](https://000004.awsstudygroup.com) |
| 5 | Amazon EBS | Tạo, gắn kết (attach), định dạng ổ cứng EBS vào EC2. Chụp Snapshot dự phòng | 17/09/2026 | 17/09/2026 | [Amazon EBS](https://docs.aws.amazon.com/ebs/) |
| 6 | Amazon S3 | Tạo Bucket, upload file, cấu hình quyền truy cập public/private | 18/09/2026 | 18/09/2026 | [Amazon S3](https://000057.awsstudygroup.com) |
| 7 | Lab Tổng hợp 1 | Dựng website tĩnh trên S3 và chạy web server cơ bản trên EC2 | 19/09/2026 | 19/09/2026 | |

### Kết quả đạt được tuần 1

- Tạo tài khoản AWS thành công, bật MFA bảo mật cho tài khoản root, tạo IAM User riêng cho lab, cấu hình AWS CLI kết nối thành công.
- Tạo Budget `Lab-Monthly-Limit` giới hạn \$10/tháng với cảnh báo khi chi tiêu vượt 10%.
- Khởi chạy EC2 instance `Lab-Server` (t2.micro, Amazon Linux 2023), kết nối SSH thành công, cấp Elastic IP cố định.
- Tạo và gắn EBS volume 2GiB (gp3) vào EC2, định dạng xfs, mount vào `/data_lab`, chụp Snapshot dự phòng thành công.
- Tạo S3 Bucket `s3-bucket-lab-fptu`, upload file, thực hành cấu hình quyền public/private.
- Hoàn thành Lab tổng hợp: website tĩnh chạy trên S3 và web server trên EC2.

---

#### Bài tập 1: Thiết lập tài khoản AWS & Bảo mật

Đăng nhập vào **AWS Management Console** với tài khoản `Manhdung2507`. Giao diện Console Home hiển thị các dịch vụ đã sử dụng gần đây: EC2, Billing, CloudWatch, IAM, Aurora/RDS, Lambda, Amazon Bedrock. Widget **Cost and usage** hiển thị tín dụng còn lại `$199.93 USD`.

> **Screenshot:** ![AWS Management Console](/images/1-Worklog/week-01/01-aws-console.png)

> **Screenshot:** ![AWS Cost and Usage Dashboard](/images/1-Worklog/week-01/02-aws-cost-dashboard.png)

Kích hoạt **MFA (Multi-Factor Authentication)** cho tài khoản root tại IAM → Security credentials. Đã gán 1 thiết bị MFA ảo tên `PhoneDung-MFA` (tạo ngày 22/09/2026). MFA giúp bảo vệ tài khoản ngay cả khi mật khẩu bị lộ.

> **Screenshot:** ![MFA Device đã được gán thành công](/images/1-Worklog/week-01/03-mfa-setup.png)

Tạo **Access Key** cho IAM User `StudentSE194349` để cấu hình AWS CLI. Lưu ý: Secret Access Key chỉ hiển thị **một lần duy nhất** khi tạo — phải lưu lại ngay.

> **Screenshot:** ![Tạo Access Key cho IAM User](/images/1-Worklog/week-01/04-iam-access-key.png)

Cấu hình AWS CLI trên máy local bằng lệnh `aws configure`. Kiểm tra kết nối bằng `aws sts get-caller-identity` — kết quả trả về đúng Account ID và ARN của user `StudentSE194349`.

```bash
# Cấu hình AWS CLI
aws configure
# AWS Access Key ID: AKIAW2FHLQZTAZ5FD2V7
# AWS Secret Access Key: ****
# Default region: us-east-1
# Default output format: json

# Kiểm tra kết nối
aws sts get-caller-identity
# Output:
# {
#   "UserId": "AIDAW2FHLQZTO3MK6ZNBK",
#   "Account": "468502414950",
#   "Arn": "arn:aws:iam::468502414950:user/StudentSE194349"
# }

# Test thử lệnh S3
aws s3 ls
```

> **Screenshot:** ![AWS CLI cấu hình và test kết nối thành công](/images/1-Worklog/week-01/05-aws-cli-config.png)

IAM User `StudentSE194349` được gán **2 policy**:
- `AmazonEC2ReadOnlyAccess` — kế thừa qua Group `Lab-Admins`
- `AmazonS3ReadOnlyAccess` — gán trực tiếp (Directly)

> **Screenshot:** ![IAM User Permissions Policies](/images/1-Worklog/week-01/06-iam-policy.png)

---

#### Bài tập 2: Cấu hình AWS Budgets

Tạo Budget tên **`Lab-Monthly-Limit`** với ngân sách `$10.00/tháng`. Budget giúp theo dõi và kiểm soát chi phí AWS, tự động gửi cảnh báo khi chi tiêu vượt ngưỡng đã đặt.

> **Screenshot:** ![Budget Lab-Monthly-Limit đã tạo thành công](/images/1-Worklog/week-01/07-budget-created.png)

Cấu hình **Alert #1**: Cảnh báo khi chi phí thực tế (`Actual`) vượt **10% ngân sách** (`$1.00`). Thông báo được gửi qua Email, AWS SNS, và AWS Chatbot. Biểu đồ Budget preview hiển thị đường ngân sách `$10.00` so với chi phí thực tế từ Sep 2025 → Sep 2026.

> **Screenshot:** ![Chi tiết Budget và cấu hình Alert](/images/1-Worklog/week-01/08-budget-detail.png)

---

#### Bài tập 3: Khởi chạy Amazon EC2

Khởi chạy EC2 instance với cấu hình:

- **Tên:** `Lab-Sever`
- **AMI:** Amazon Linux 2023
- **Instance type:** `t2.micro` (Free Tier eligible)
- **Instance ID:** `i-08023703be10a8d23`
- **Public IPv4:** `54.147.198.241`
- **Private IPv4:** `172.31.92.66`
- **Availability Zone:** `us-east-1b`
- **Trạng thái:** ✅ Running

> **Screenshot:** ![EC2 Instance đang chạy - Trạng thái Running](/images/1-Worklog/week-01/09-ec2-running.png)

Kết nối EC2 qua **EC2 Instance Connect** trên web browser (chọn Public subnet access với Managed SSH keys). Instance có Status check 2/2 passed, đảm bảo instance hoạt động bình thường.

> **Screenshot:** ![Giao diện Connect to Linux instance](/images/1-Worklog/week-01/10-ec2-connect.png)

Kết nối SSH thành công vào EC2 instance từ máy local bằng key pair `EC2-Lab.pem`. Terminal hiển thị banner **Amazon Linux 2023** và dấu nhắc lệnh `[ec2-user@ip-172-31-26-62 ~]$`.

```bash
# Kết nối SSH vào EC2
ssh -i "EC2-Lab.pem" ec2-user@100.58.113.158

# Kết quả thành công:
#        #_
#  ~\_  ####_        Amazon Linux 2023
# ~~  \_#####\
# ~~     \###|       https://aws.amazon.com/linux/amazon-linux-2023
# ~~       \#/ ___
# ~~       V~' '->
# ~~~         /
#  ~~._.   _/
#     _/ _/
#   _/m/'
# [ec2-user@ip-172-31-26-62 ~]$
```

> **Screenshot:** ![SSH kết nối thành công vào EC2](/images/1-Worklog/week-01/11-ssh-connected.png)

Cấp phát **Elastic IP** `3.211.17.203` (Public IP cố định) để EC2 giữ nguyên địa chỉ IP khi restart. Elastic IP được cấp từ pool Amazon, Scope: VPC, Region: `us-east-1`.

> **Screenshot:** ![Elastic IP đã được cấp phát](/images/1-Worklog/week-01/12-elastic-ip.png)

---

#### Bài tập 4: Amazon EBS — Tạo, gắn kết và định dạng ổ đĩa

Tạo EBS Volume với thông số:

- **Volume ID:** `vol-01b6945e9ce043db8`
- **Size:** 2 GiB
- **Type:** gp3 (General Purpose SSD)
- **IOPS:** 3000 | **Throughput:** 125
- **Availability Zone:** `us-east-1c`
- **Volume state:** ✅ In-use
- **Attached:** `i-05441fcc8d06a176 (EC2-Lab)` tại `/dev/sdb`

> **Screenshot:** ![EBS Volume đã tạo và gắn vào EC2](/images/1-Worklog/week-01/13-ebs-volume.png)

Định dạng và mount EBS volume vào EC2. Sau khi gắn volume, thực hiện format xfs và mount vào `/data_lab`. Lệnh `df -h` xác nhận `/dev/xvdb` (2.0G) đã được mount thành công vào `/data_lab` với 3% dung lượng đã dùng.

```bash
# Format ổ đĩa xfs
sudo mkfs -t xfs /dev/xvdb

# Tạo thư mục mount point
sudo mkdir /data_lab

# Mount volume
sudo mount /dev/xvdb /data_lab

# Kiểm tra
df -h
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/xvda1      8.0G  1.6G  6.4G  20% /
# /dev/xvdb       2.0G   47M  1.9G   3% /data_lab
```

> **Screenshot:** ![EBS mount thành công - df -h hiển thị /data_lab](/images/1-Worklog/week-01/14-ebs-mount.png)

Chụp **EBS Snapshot** để dự phòng dữ liệu:

- **Snapshot ID:** `snap-0d1423504beac3d88`
- **Description:** "Ban Sao O Cung 2GB"
- **Full snapshot size:** 66.5 MiB | **Volume size:** 2 GiB
- **Snapshot status:** ✅ Completed (30/09/2026)

> **Screenshot:** ![EBS Snapshot đã hoàn thành](/images/1-Worklog/week-01/15-ebs-snapshot.png)

---

#### Bài tập 5: Amazon S3 — Quản lý Bucket và quyền truy cập

Upload file lên S3 Bucket `s3-bucket-lab-fptu` thành công (Upload succeeded). Cấu hình **Default encryption** dùng SSE-S3 (Server-side encryption với Amazon S3 managed keys) — mã hóa tự động áp dụng cho toàn bộ object mới.

> **Screenshot:** ![S3 Upload thành công - Cấu hình Default Encryption](/images/1-Worklog/week-01/16-s3-upload-success.png)

Bucket `s3-bucket-lab-fptu` chứa 1 object: `Screenshot 2026-09-15 122113.png` (40.3 KB, Storage class: Standard). Bucket có đầy đủ các tab quản lý: Objects, Metadata, Properties, Permissions, Metrics, Management.

> **Screenshot:** ![S3 Bucket với file đã upload](/images/1-Worklog/week-01/17-s3-bucket.png)

Khi chưa cấu hình quyền public, truy cập trực tiếp URL object bị từ chối với lỗi **AccessDenied** (403). Đây là hành vi mặc định bảo mật của S3 — mọi object đều private cho đến khi được cấp quyền public.

> **Screenshot:** ![S3 trả về lỗi Access Denied khi chưa cấp quyền public](/images/1-Worklog/week-01/18-s3-access-denied.png)

Thực hiện **Make public** cho object `Screenshot 2026-09-15 122113.png` thông qua ACL. Lưu ý quan trọng: Khi public read access được bật và không bị chặn bởi Block Public Access settings, **bất kỳ ai trên internet đều có thể truy cập** object này.

> **Screenshot:** ![Cấu hình Make public cho S3 object](/images/1-Worklog/week-01/19-s3-make-public.png)

---

#### Bài tập 6: Lab Tổng hợp 1

**Phần A: Website tĩnh trên S3**

Tạo file `index.html` với nội dung: *"Xin chao, day la website tinh tren AWS S3! He thong phong thu mang dang hoat dong tot."* Upload lên S3 và bật **Static Website Hosting**. Website truy cập thành công qua S3 endpoint URL.

> **Screenshot:** ![Website tĩnh chạy thành công trên S3](/images/1-Worklog/week-01/21-s3-static-website.png)

**Phần B: Web Server trên EC2**

Cài đặt web server trên EC2 và tạo file HTML: *"Chao mung ban den voi Web Server tren EC2! Lab FPTU hoan thanh xuat sac!"*

Để EC2 có thể phục vụ web, cần cấu hình **Security Group** với Inbound rules:
- SSH (TCP port 22) — Source: `0.0.0.0/0`
- HTTP (TCP port 80) — Source: `0.0.0.0/0`

> **Screenshot:** ![Security Group cấu hình Inbound Rules SSH + HTTP](/images/1-Worklog/week-01/23-security-group.png)

Ban đầu truy cập EC2 qua IP `100.58.113.158` bị **timeout** do Security Group chưa mở port 80. Sau khi cập nhật Inbound rules cho HTTP, web server truy cập thành công.

> **Screenshot:** ![Lỗi timeout trước khi mở port 80](/images/1-Worklog/week-01/22-ec2-web-server.png)

> **Screenshot:** ![Web Server EC2 chạy thành công qua browser](/images/1-Worklog/week-01/24-ec2-web-success.png)

---

#### Khó khăn gặp phải

| Vấn đề | Cách giải quyết |
| :--- | :--- |
| S3 object trả về `AccessDenied` (403) dù đã upload | Cần thực hiện **Make public** qua ACL hoặc thêm Bucket Policy cho object |
| EC2 web server bị timeout khi truy cập qua browser | Security Group chưa mở **Inbound port 80 (HTTP)** — thêm rule HTTP `0.0.0.0/0` |
| SSH port 22 mở cho `0.0.0.0/0` (toàn internet) | Nên đổi source thành **My IP** để hạn chế truy cập SSH từ địa chỉ không xác định |
| EBS volume không thấy sau khi attach | Cần chạy `lsblk` để kiểm tra, hệ thống cần vài giây để nhận diện thiết bị mới |

### Kế hoạch tuần 2

- Tìm hiểu VPC: Subnet, Route Table, Internet Gateway, NAT Gateway.
- Tạo VPC tùy chỉnh với public/private subnet.
- Triển khai EC2 bên trong VPC.
- Tìm hiểu Load Balancer và Auto Scaling Group.
