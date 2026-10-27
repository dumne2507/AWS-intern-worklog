---
title: "Worklog Tuần 1"
date: 2026-09-14
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

## Mục tiêu tuần 1: Khởi tạo tài khoản & Dịch vụ cơ bản

- **Tài khoản AWS & Bảo mật:** Khởi tạo tài khoản AWS Free Tier, kích hoạt xác thực đa yếu tố (MFA) cho tài khoản root và cấu hình kết nối AWS CLI an toàn.
- **AWS IAM & Budgets:** Thiết lập người dùng, nhóm quyền IAM User/Group và tạo cảnh báo ngân sách chi phí hàng tháng với AWS Budgets.
- **Amazon EC2:** Khởi chạy máy chủ ảo EC2 Linux, kết nối từ xa qua SSH và cấu hình Elastic IP cố định.
- **Amazon EBS:** Quản lý khối lưu trữ ổ đĩa: tạo volume gp3, gắn kết (attach), định dạng hệ thống tệp tin xfs và tạo bản sao lưu Snapshot dự phòng.
- **Amazon S3:** Khởi tạo kho lưu trữ đối tượng S3 Bucket, tải lên tệp tin và thiết lập phân quyền truy cập công khai/nội bộ (Public/Private Access).
- **Lab Tổng hợp 1:** Triển khai website tĩnh trên S3 Static Website Hosting và dựng máy chủ web server Apache trên EC2.

### Các công việc cần triển khai trong tuần này

| Thứ | Chủ đề | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **2** | **Tài khoản AWS & Bảo mật** | Khởi tạo AWS, kích hoạt bảo mật MFA, tạo IAM User và cấu hình AWS CLI | 14/09/2026 | 14/09/2026 | [AWS Free Tier](https://aws.amazon.com/free/) |
| **3** | **IAM & AWS Budgets** | Phân quyền IAM User/Group, thiết lập cảnh báo ngân sách chi phí (AWS Budgets) | 15/09/2026 | 15/09/2026 | [AWS IAM](https://000002.awsstudygroup.com) |
| **4** | **Amazon EC2** | Khởi chạy máy chủ ảo EC2, tạo Key Pair, kết nối SSH và cấp phát Elastic IP | 16/09/2026 | 16/09/2026 | [Amazon EC2](https://000004.awsstudygroup.com) |
| **5** | **Amazon EBS** | Tạo ổ đĩa EBS gp3, gắn kết (attach), format xfs và tạo Snapshot dự phòng | 17/09/2026 | 17/09/2026 | [Amazon EBS](https://docs.aws.amazon.com/ebs/) |
| **6** | **Amazon S3** | Tạo S3 Bucket, upload tệp tin, cấu hình quyền truy cập public/private | 18/09/2026 | 18/09/2026 | [Amazon S3](https://000057.awsstudygroup.com) |
| **7** | **Lab Tổng hợp 1** | Dựng website tĩnh trên S3 và triển khai web server hoàn chỉnh trên EC2 | 19/09/2026 | 19/09/2026 | [AWS Architecture](https://aws.amazon.com/architecture/) |

### Kết quả đạt được tuần 1

- Tạo tài khoản AWS thành công, bật MFA bảo mật cho tài khoản root, tạo IAM User riêng cho bài lab, cấu hình AWS CLI kết nối thành công.
- Tạo Budget `Lab-Monthly-Limit` giới hạn \$10/tháng với cảnh báo tự động khi chi tiêu thực tế vượt 10%.
- Khởi chạy EC2 instance `Lab-Server` (t2.micro, Amazon Linux 2023), kết nối SSH thành công, cấp Elastic IP cố định.
- Tạo và gắn EBS volume 2GiB (gp3) vào EC2, định dạng xfs, mount vào `/data_lab`, chụp Snapshot dự phòng thành công.
- Tạo S3 Bucket `s3-bucket-lab-fptu`, upload file, thực hành cấu hình quyền public/private an toàn.
- Hoàn thành xuất sắc Lab tổng hợp 1: website tĩnh chạy trên S3 và web server chạy trên EC2.

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

#### Bài tập 6: Lab Tổng hợp 1 — Website tĩnh trên S3 & Web Server trên EC2

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

#### Khó khăn gặp phải & Cách giải quyết

| Vấn đề | Nguyên nhân | Cách giải quyết |
| :--- | :--- | :--- |
| S3 object trả về lỗi `AccessDenied` (403) khi truy cập URL | S3 mặc định bật tính năng Block Public Access và mọi object đều ở chế độ private | Tắt Block Public Access ở cấp Bucket (nếu phục vụ static website công khai) và thực hiện **Make public** qua ACL hoặc gắn Bucket Policy |
| EC2 web server bị timeout khi truy cập qua trình duyệt | Security Group của EC2 chưa mở cổng tiếp nhận lưu lượng HTTP | Bổ sung Inbound Rule mở cổng **HTTP (port 80)** với Source `0.0.0.0/0` vào Security Group của instance |
| Cổng SSH (port 22) mở cho `0.0.0.0/0` tiềm ẩn nguy cơ dò quét | Cấu hình mặc định mở SSH cho toàn bộ dải IP trên Internet | Giới hạn Source của SSH rule về **My IP** để chỉ cho phép địa chỉ IP hiện tại của quản trị viên kết nối |
| Ổ đĩa EBS không hiển thị trong lệnh `df -h` sau khi gắn kết | Ổ đĩa mới gắn kết chưa được format hệ thống tệp và chưa mount vào thư mục | Kiểm tra tên thiết bị bằng `lsblk`, sau đó chạy `sudo mkfs -t xfs /dev/xvdb` để format và `sudo mount /dev/xvdb /data_lab` |

### Kế hoạch tuần 2: Mạng & CDN

- Khởi tạo Virtual Private Cloud (VPC), cấu hình dải IPv4 CIDR và phân tách Public/Private Subnet.
- Cấu hình bảng định tuyến (Route Table), Internet Gateway (IGW) và NAT Gateway cho Private Subnet.
- Quản lý dịch vụ DNS toàn cầu với Amazon Route 53 và trỏ bản ghi A Record.
- Tối ưu hóa phân phối nội dung tĩnh qua Amazon CloudFront CDN kết nối S3 Origin.
- Thực hành điện toán biên với Lambda@Edge và thiết lập Security Groups bảo vệ mạng nội bộ.
