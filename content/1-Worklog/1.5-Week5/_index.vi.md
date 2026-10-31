---
title: "Worklog Tuần 5"
date: 2026-10-12
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

## Mục tiêu tuần 5: Bảo mật hệ thống

- **AWS WAF:** Khởi tạo Web Application Firewall liên kết với Application Load Balancer (ALB), xây dựng các bộ quy tắc (Web ACL) ngăn chặn tấn công SQL Injection và Cross-Site Scripting (XSS).
- **AWS GuardDuty:** Kích hoạt dịch vụ phát hiện mối đe dọa thông minh, phân tích nguồn dữ liệu log nhằm nhận diện các hành vi bất thường như quét cổng (Portscan) và giao tiếp máy chủ điều khiển (C&C Server).
- **VPC Flow Logs:** Thiết lập giám sát luồng lưu lượng mạng toàn diện tại VPC/Subnet, lưu trữ log vào S3 Bucket để điều tra, phát hiện các kết nối bị từ chối (REJECT) từ địa chỉ IP độc hại.
- **AWS Security Hub & AWS Config:** Kích hoạt Security Hub đánh giá điểm tuân thủ an ninh đám mây (CSPM Score) theo chuẩn CIS AWS Foundations Benchmark và thiết lập bộ ghi cấu hình với AWS Config.
- **Mã hóa dữ liệu với AWS KMS:** Tạo Customer Managed Key (CMK), áp dụng mã hóa dữ liệu lưu trữ (Encryption at Rest) cho khối lưu trữ ổ đĩa EBS Volume và kho lưu trữ S3 Bucket với tính năng Bucket Key.
- **Lab Tổng hợp 5:** Thực hiện giả lập tấn công an ninh mạng thực tế (dùng Nmap quét cổng và chèn mã độc SQLi qua curl/browser), kiểm chứng khả năng phát hiện tự động của GuardDuty và cơ chế chặn tức thời của AWS WAF.

### Các công việc cần triển khai trong tuần này

| Thứ | Chủ đề | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **2** | **AWS WAF** | Cấu hình Web ACL trên ALB, thiết lập quy tắc chặn SQL Injection và XSS | 12/10/2026 | 12/10/2026 | [AWS WAF](https://docs.aws.amazon.com/waf/) |
| **3** | **AWS GuardDuty** | Kích hoạt GuardDuty, phân tích các phát hiện bất thường C&C Activity và Portscan | 13/10/2026 | 13/10/2026 | [AWS GuardDuty](https://docs.aws.amazon.com/guardduty/) |
| **4** | **VPC Flow Logs** | Bật Flow Logs cho VPC/Subnet, đẩy log nén về S3 Bucket và phân tích gói tin REJECT | 14/10/2026 | 14/10/2026 | [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html) |
| **5** | **Security Hub & Config** | Kích hoạt Security Hub, đo điểm tư thế bảo mật CSPM và cấu hình AWS Config Recorder | 15/10/2026 | 15/10/2026 | [AWS Security Hub](https://docs.aws.amazon.com/securityhub/) |
| **6** | **Mã hóa dữ liệu (KMS)** | Tạo Customer Managed Key (`lab-kms-key-labt6`), mã hóa EBS Volume và S3 Bucket | 16/10/2026 | 16/10/2026 | [AWS KMS](https://docs.aws.amazon.com/kms/) |
| **7** | **Lab Tổng hợp 5** | Dựng WAF Protection Pack, dùng Nmap quét cổng và curl tấn công SQLi, phân tích kết quả | 17/10/2026 | 17/10/2026 | [AWS Threat Detection](https://aws.amazon.com/security/) |

### Kết quả đạt được tuần 5

- Triển khai thành công AWS WAF bảo vệ Application Load Balancer `labt2-waf-alb-1192945120.us-east-1.elb.amazonaws.com`, chặn đứng 100% các request chèn mã SQLi (`?username=admin' OR '1'='1`) và XSS (`?search=<script>alert(1)</script>`) với mã phản hồi `403 Forbidden`.
- Kích hoạt AWS GuardDuty tại `us-east-1`, phát hiện chính xác các mối đe dọa nghiêm trọng trên instance `i-034f1dad8608f0a48` gồm `Backdoor:EC2/C&CActivity.B!DNS` (Mức độ High) và `Recon:EC2/Portscan` (Mức độ Medium).
- Thiết lập VPC Flow Logs `fl-0626508efb0473611` trỏ về S3 Bucket `labt4-vpc-flowlogs`, trích xuất và phân tích thành công log mạng chỉ ra các đợt quét cổng nguy hiểm từ các IP lạ (`85.11.167.89`, `216.180.246.118`, `192.161.49.2`) với trạng thái `REJECT OK`.
- Kích hoạt thành công AWS Security Hub đạt điểm đánh giá tư thế an ninh mạng CSPM ban đầu là 30% (7 Passed, 16 Failed trên 377 tiêu chí), đồng thời cấu hình AWS Config theo dõi thay đổi cấu hình tài nguyên.
- Tạo thành công Customer Managed Key `lab-kms-key-labt6` (`e7ae1582-7e6f-4de0-8ff0-b5aadb9ac466`), mã hóa ổ đĩa EBS 1 GiB gp3 `vol-0da2c452479eef183` và cấu hình Default Encryption chuẩn SSE-KMS kèm Bucket Key trên S3 Bucket `labt6-s3-for-kms`.
- Hoàn thành xuất sắc Lab Tổng hợp 5: cấu hình WAF Essential Protection Pack, cài đặt Nmap trên máy chủ `LabT7-WAF` (`i-0584ad61366a505b4`), thực hiện quét cổng từ xa kích hoạt GuardDuty nhận diện `c0d0beb9531b554182419977d31f37e8`, đồng thời kiểm chứng WAF chặn đứng truy vấn SQL Injection gửi tới ALB `LabT7-ALB`.

---

#### Bài tập 1: AWS WAF — Cấu hình Web ACL Bảo vệ Khỏi SQL Injection & XSS

Triển khai một kiến trúc bảo vệ tầng ứng dụng (Layer 7) hoàn chỉnh: Khởi chạy một Web Server máy chủ ảo EC2 và đưa vào phía sau Application Load Balancer mang tên **`labt2-waf-alb`**:
- **Địa chỉ IP máy chủ Web gốc (Origin):** `44.222.158.206`
- **DNS Endpoint của Load Balancer:** `labt2-waf-alb-1192945120.us-east-1.elb.amazonaws.com`
- **Vùng hoạt động:** `us-east-1` (N. Virginia)

Khi truy cập trực tiếp bằng địa chỉ IP của máy chủ web thông thường, ứng dụng phản hồi mượt mà với thông điệp: *"Hello from AWS WAF Lab Web Server"*.

> **Screenshot:** ![Truy cập máy chủ web gốc phản hồi bình thường](/images/1-Worklog/week-05/01-waf-web-server-origin.png)

Tiến hành khởi tạo **Web ACL** trên dịch vụ **AWS WAF** và liên kết trực tiếp với ALB `labt2-waf-alb`. Cấu hình 2 nhóm quy tắc bảo mật cốt lõi:
- **AWSManagedRulesSQLiRuleSet:** Bộ quy tắc phòng vệ các mẫu tấn công SQL Injection vào URI, query string và body.
- **AWSManagedRulesCommonRuleSet:** Bộ quy tắc phòng vệ các mẫu mã độc Cross-Site Scripting (XSS) và đầu vào bất thường.

Tiến hành kiểm thử giả lập tấn công qua đường dẫn ALB:
- **Kiểm thử tấn công SQL Injection:** Gửi truy vấn chứa câu lệnh bypass authentication điển hình: `http://labt2-waf-alb-1192945120.us-east-1.elb.amazonaws.com/?username=admin%27%20OR%20%271%27=%271`. Hệ thống WAF lập tức can thiệp và ngắt kết nối, trình duyệt trả về mã lỗi **`403 Forbidden`**.

> **Screenshot:** ![AWS WAF chặn đứng tấn công SQL Injection trả về mã 403 Forbidden](/images/1-Worklog/week-05/02-waf-block-sql-injection.png)

- **Kiểm thử tấn công XSS:** Gửi chuỗi script độc hại qua tham số tìm kiếm: `http://labt2-waf-alb-1192945120.us-east-1.elb.amazonaws.com/?search=<script>alert(1)</script>`. Bộ quy tắc Common Rule Set của WAF nhận diện ngay đoạn script nguy hiểm và lập tức chặn đứng với mã lỗi **`403 Forbidden`**.

> **Screenshot:** ![AWS WAF chặn đứng tấn công XSS trả về mã 403 Forbidden](/images/1-Worklog/week-05/03-waf-block-xss.png)

---

#### Bài tập 2: AWS GuardDuty — Kích hoạt Phát hiện Mối đe dọa & Phân tích Log Bất thường

Bật dịch vụ **AWS GuardDuty** trên tài khoản `468502414950` (`Manhdung2507`) tại Region `us-east-1`. GuardDuty hoạt động liên tục mà không cần cài đặt agent, tự động thu thập và phân tích dữ liệu từ VPC Flow Logs, DNS Logs và CloudTrail Management Events bằng Machine Learning.

Sau một khoảng thời gian theo dõi, trang **Findings** của GuardDuty ghi nhận danh sách các phát hiện an ninh nghiêm trọng đối với máy chủ EC2 **`i-034f1dad8608f0a48`**:

> **Screenshot:** ![Danh sách phát hiện an ninh trên giao diện AWS GuardDuty](/images/1-Worklog/week-05/04-guardduty-findings-list.png)

Phân tích chi tiết các mối nguy hiểm được GuardDuty cảnh báo:
- **Mối đe dọa C&C Activity (Mức độ High):**
  - **Tiêu đề:** `The EC2 instance i-034f1dad8608f0a48 queried a known Command & Control server domain name.`
  - **Finding ID:** `a6d0c1a1a01159c61fedac2dcccde50d`
  - **Loại phát hiện:** `Backdoor:EC2/C&CActivity.B!DNS`
  - **Mức độ nghiêm trọng (Severity):** **HIGH** (Đỏ)
  - **Số lần ghi nhận (Count):** `4`
  - **Resource ID:** `i-034f1dad8608f0a48`
  - **Trạng thái quét mã độc (Malware scan):** `RUNNING` (Scan ID: `4111d46d648491294be6afb8f7379081`)
  - **Ý nghĩa bảo mật:** Máy chủ đã thực hiện các truy vấn DNS ra ngoài tới một domain thuộc danh sách máy chủ điều khiển botnet (C&C Server), dấu hiệu cho thấy máy đã bị nhiễm mã độc backdoor.

> **Screenshot:** ![Chi tiết phát hiện Backdoor C&C Activity trên instance](/images/1-Worklog/week-05/05-guardduty-finding-cnc-detail.png)

- **Mối đe dọa Outbound Portscan (Mức độ Medium):**
  - **Tiêu đề:** `An outbound portscan was detected from EC2 instance i-034f1dad8608f0a48.`
  - **Loại phát hiện:** `Recon:EC2/Portscan`
  - **Mức độ nghiêm trọng (Severity):** **MEDIUM** (Cam)
  - **Số lần ghi nhận (Count):** `5`
  - **Ý nghĩa bảo mật:** Instance bị lợi dụng làm bàn đạp (pivot) để thực hiện các đợt quét cổng dò tìm lỗ hổng mạng hướng ra các mục tiêu bên ngoài.

> **Screenshot:** ![Chi tiết phát hiện hành vi quét cổng outbound từ instance](/images/1-Worklog/week-05/06-guardduty-finding-portscan.png)

---

#### Bài tập 3: VPC Flow Logs — Giám sát Lưu lượng Mạng & Điều tra An ninh

Thiết lập tính năng **VPC Flow Logs** để ghi lại toàn bộ siêu dữ liệu (metadata) của các gói tin IP đi vào và đi ra khỏi mạng VPC:
- **Flow Log ID:** `fl-0626508efb0473611`
- **Phạm vi giám sát (Scope):** Toàn bộ giao diện mạng (ENI) trong VPC
- **Lưu lượng giám sát:** `ALL` (Bao gồm cả traffic ACCEPT và REJECT)
- **Đích lưu trữ (Destination):** S3 Bucket mang tên **`labt4-vpc-flowlogs`**

Các tệp log được hệ thống tự động nén dưới định dạng `.log.gz` và phân cấp theo cấu trúc: `AWSLogs/468502414950/vpcflowlogs/us-east-1/2026/10/30/`.

> **Screenshot:** ![Kho lưu trữ S3 Bucket chứa các tệp nén VPC Flow Logs](/images/1-Worklog/week-05/07-vpc-flowlogs-s3-bucket.png)

Tải xuống tệp log `468502414950_vpcflowlogs_us-east-1_fl-0626508efb0473611_...log.gz`, giải nén và mở bằng trình soạn thảo để tiến hành phân tích lưu lượng:

```
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
2 468502414950 eni-08f0d58b5d35c8399 85.11.167.89 172.31.5.159 48994 3001 6 1 40 1793369986 1793370024 REJECT OK
2 468502414950 eni-08f0d58b5d35c8399 216.180.246.118 172.31.5.159 21413 8500 6 4 176 1793369986 1793370024 REJECT OK
2 468502414950 eni-08f0d58b5d35c8399 192.161.49.2 172.31.5.159 44718 9215 6 1 40 1793369986 1793370024 REJECT OK
2 468502414950 eni-08f0d58b5d35c8399 193.163.125.176 172.31.5.159 47360 8443 6 1 44 1793369986 1793370024 REJECT OK
```

**Phân tích phát hiện an ninh từ log:**
- **Interface bị nhắm tới:** `eni-08f0d58b5d35c8399` (Địa chỉ IP nội bộ: `172.31.5.159`)
- **Các địa chỉ IP nguồn lạ:** `85.11.167.89`, `216.180.246.118`, `192.161.49.2`, `193.163.125.176`
- **Các cổng đích bị rà quét:** Cổng `3001`, `8500`, `9215`, `8443` (Protocol `6` - TCP)
- **Hành động (Action):** **`REJECT OK`** — Toàn bộ các kết nối dò quét này đã bị Security Group và Network ACL chặn lại thành công tại lớp biên của VPC.

> **Screenshot:** ![Phân tích chi tiết nội dung tệp tin VPC Flow Log với hành động REJECT](/images/1-Worklog/week-05/08-vpc-flowlogs-analysis-reject.png)

---

#### Bài tập 4: AWS Security Hub & AWS Config — Đánh giá Tuân thủ Chuẩn CIS

Kích hoạt dịch vụ **AWS Security Hub** để thiết lập trung tâm tổng hợp thông tin an ninh tập trung, quét và đánh giá tư thế an ninh mạng đám mây (Cloud Security Posture Management - CSPM) theo chuẩn tiêu chuẩn an ninh **CIS AWS Foundations Benchmark**:
- **Bảng điều khiển Posture Management:**
  - **Điểm số tuân thủ tổng thể (CSPM Score):** **`30%`**
  - **Tiêu chí đạt (Passed):** `7` tiêu chuẩn an ninh
  - **Tiêu chí chưa đạt (Failed):** `16` tiêu chuẩn cần khắc phục
  - **Chưa có dữ liệu (No data):** `377` tiêu chí
- **Đánh giá sơ bộ:** Điểm số 30% phản ánh môi trường lab mới khởi tạo, còn thiếu các cấu hình tuân thủ nâng cao như: bắt buộc xoay vòng khóa KMS định kỳ, cấu hình CloudTrail đa vùng và bật kiểm toán ghi log S3.

> **Screenshot:** ![Điểm số đánh giá tư thế bảo mật CSPM trên AWS Security Hub](/images/1-Worklog/week-05/09-security-hub-posture-score.png)

Song song đó, kích hoạt dịch vụ **AWS Config** để liên tục theo dõi, ghi lại các thay đổi trạng thái cấu hình của toàn bộ tài nguyên trên tài khoản AWS:
- **Trình ghi cấu hình (Configuration Recorder):** Bật chế độ tự động ghi nhận các Configuration Items (CI) cho các tài nguyên EC2, S3, IAM và Security Groups.
- **Conformance Packs & Rules:** Chuẩn bị triển khai các bộ quy tắc tự động đánh giá tuân thủ (Compliance status) phục vụ kiểm toán bảo mật liên tục.

> **Screenshot:** ![Bảng điều khiển AWS Config Dashboard theo dõi trạng thái tuân thủ](/images/1-Worklog/week-05/10-aws-config-dashboard.png)

---

#### Bài tập 5: Mã hóa Dữ liệu (AWS KMS) — Tạo CMK Mã hóa EBS Volume & S3 Bucket

Khởi tạo một khóa mã hóa do người dùng tự quản lý (**Customer Managed Key - CMK**) trên dịch vụ **AWS Key Management Service (KMS)** mang định danh:
- **KMS Key Alias:** **`lab-kms-key-labt6`**
- **KMS Key ID:** `e7ae1582-7e6f-4de0-8ff0-b5aadb9ac466`
- **KMS Key ARN:** `arn:aws:kms:us-east-1:468502414950:key/e7ae1582-7e6f-4de0-8ff0-b5aadb9ac466`
- **Vùng tạo khóa:** `us-east-1` (US East - N. Virginia)

Ứng dụng khóa CMK vừa tạo vào việc bảo vệ dữ liệu lưu trữ tại 2 tầng dịch vụ chính:
- **Mã hóa kho lưu trữ S3 Bucket:** Tạo S3 Bucket mới mang tên **`labt6-s3-for-kms`** và cấu hình chính sách mã hóa mặc định (**Default encryption**):
  - **Loại mã hóa (Encryption type):** `Server-side encryption with AWS Key Management Service keys (SSE-KMS)`
  - **Khóa KMS chỉ định:** `arn:aws:kms:us-east-1:468502414950:key/e7ae1582-7e6f-4de0-8ff0-b5aadb9ac466`
  - **Tính năng Bucket Key:** `Enabled` (Tối ưu hóa chi phí bằng cách giảm thiểu tới 99% các cuộc gọi giải mã trực tiếp tới KMS)
  - **Chặn loại mã hóa không an toàn:** `Server-Side Encryption with Customer-Provided Keys (SSE-C)`

> **Screenshot:** ![Cấu hình mã hóa mặc định SSE-KMS trên S3 Bucket labt6-s3-for-kms](/images/1-Worklog/week-05/11-kms-s3-default-encryption.png)

- **Mã hóa khối lưu trữ ổ đĩa EBS Volume:** Khởi tạo một EBS Volume mới với các thông số chi tiết:
  - **Volume ID:** `vol-0da2c452479eef183`
  - **Dung lượng:** `1 GiB` | **Loại ổ cứng:** `gp3` (General Purpose SSD)
  - **IOPS:** `3000` | **Throughput:** `125`
  - **Availability Zone:** `use1-az1` (`us-east-1a`)
  - **Trạng thái mã hóa (Encryption):** `Encrypted`
  - **KMS Key Alias liên kết:** `lab-kms-key-labt6`
  - **Trạng thái ổ đĩa:** ✅ `Available` | **Kiểm tra trạng thái:** `Okay`

> **Screenshot:** ![EBS Volume được mã hóa an toàn bằng Customer Managed Key](/images/1-Worklog/week-05/12-kms-ebs-volume-encrypted.png)

---

#### Bài tập 6: Lab Tổng hợp 5 — Giả lập Tấn công Nmap & Kiểm tra Log Phát hiện GuardDuty/WAF

Thực hiện bài kiểm thử an ninh tổng hợp cuối tuần: Dựng môi trường mô phỏng tấn công thực tế từ bên ngoài nhằm kiểm chứng tính hiệu quả của các tầng phòng thủ WAF và GuardDuty đã triển khai.

**Bước 1: Cấu hình WAF Protection Pack (Essential Rules)**
Trên giao diện **AWS WAF & Shield**, khởi tạo một Protection Pack hoàn chỉnh tích hợp các bộ quy tắc thiết yếu:
- `Layer 7 anti-DDoS:` Count
- `IP blocklist & Geolocation restriction:` Block
- `AWS core rule set:` Default Block
- `SQL injection protection:` Default Block
- `Linux and Unix protection:` Default Block

> **Screenshot:** ![Cấu hình bộ quy tắc WAF Protection Pack Essential Rules](/images/1-Worklog/week-05/13-lab5-waf-essential-rules.png)

**Bước 2: Chuẩn bị máy chủ kiểm thử an ninh**
Khởi chạy máy chủ ảo EC2 mang tên **`LabT7-WAF`** làm môi trường kiểm thử:
- **Instance ID:** `i-0584ad61366a505b4`
- **Hệ điều hành:** Amazon Linux 2023
- **Public IPv4:** `100.58.223.95`
- **Private IPv4:** `172.31.3.123`
- Tiến hành cài đặt công cụ quét cổng mạng chuyên dụng **Nmap**:
  ```bash
  sudo dnf install nmap -y
  # Cài đặt thành công 3 gói: nmap, libssh2, nmap-ncat
  ```

> **Screenshot:** ![Cài đặt công cụ Nmap trên máy chủ EC2 LabT7-WAF](/images/1-Worklog/week-05/14-lab5-ec2-install-nmap.png)

**Bước 3: Giả lập tấn công chèn mã SQL Injection qua curl**
Từ bên trong máy chủ `LabT7-WAF`, thực hiện lệnh `curl` gửi request chứa payload SQLi tới Application Load Balancer **`LabT7-ALB`**:

```bash
[ec2-user@ip-172-31-3-123 ~]$ curl -I "http://LabT7-ALB-1504252521.us-east-1.elb.amazonaws.com/?id=1'%20OR%20'1'='1"
HTTP/1.1 403 Forbidden
Server: awselb/2.0
Date: Tue, 27 Oct 2026 09:49:06 GMT
Content-Type: text/html
Content-Length: 118
Connection: keep-alive
```

Phản hồi trả về ngay lập tức là **`HTTP/1.1 403 Forbidden`** từ tầng `awselb/2.0`, minh chứng WAF đã phân tích payload và triệt tiêu cuộc tấn công trước khi request chạm tới Web Server.

> **Screenshot:** ![Lệnh curl gửi mã độc SQLi bị WAF chặn với mã phản hồi 403 Forbidden](/images/1-Worklog/week-05/15-lab5-curl-sqli-waf-blocked.png)

**Bước 4: Thực hiện quét cổng toàn diện từ máy tính cá nhân bằng Nmap**
Từ máy tính local qua Windows PowerShell, chạy lệnh quét toàn bộ 65,535 cổng TCP đối với máy chủ EC2 `100.58.223.95`:

```powershell
PS C:\Users\Admin> nmap -Pn -A -p- -T4 100.58.223.95
Starting Nmap 7.993 ( https://nmap.org ) at 2026-10-27 19:41 +0700
Nmap scan report for ec2-100-58-223-95.compute-1.amazonaws.com (100.58.223.95)
Host is up (0.25s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 9.9 (protocol 2.0)
80/tcp  closed http
443/tcp closed https
```

> **Screenshot:** ![Thực hiện quét cổng TCP bằng Nmap từ máy tính cá nhân](/images/1-Worklog/week-05/16-lab5-powershell-nmap-scan.png)

**Bước 5: Kiểm tra kết quả phát hiện mối đe dọa trên AWS GuardDuty**
Chỉ vài phút sau khi cuộc quét cổng diễn ra, AWS GuardDuty lập tức tạo một phát hiện bảo mật mức độ Medium:
- **Tiêu đề phát hiện:** `An outbound portscan was detected from EC2 instance i-0584ad61366a505b4.`
- **Finding ID:** `c0d0beb9531b554182419977d31f37e8`
- **Loại phát hiện:** `Recon:EC2/Portscan`
- **Severity:** `MEDIUM` | **Count:** `1`
- **Chi tiết hành vi:** GuardDuty phân tích VPC Flow Logs và nhận diện chính xác máy chủ `i-0584ad61366a505b4` đang tham gia hoặc nhận các đợt quét cổng bất thường hướng tới `100.58.223.95`.

> **Screenshot:** ![AWS GuardDuty nhận diện chính xác hành vi rà quét cổng mạng](/images/1-Worklog/week-05/17-lab5-guardduty-detected-portscan.png)

**Bước 6: Kiểm chứng lại trên trình duyệt Web**
Nhập trực tiếp URL chứa chuỗi SQLi `http://labt7-alb-1504252521.us-east-1.elb.amazonaws.com/?id=1%27%20OR%20%271%27=%271` trên trình duyệt Edge. Giao diện trả về trang trắng với dòng chữ lớn duy nhất: **`403 Forbidden`**.

> **Screenshot:** ![Trình duyệt xác nhận truy cập bị chặn 403 Forbidden bởi AWS WAF](/images/1-Worklog/week-05/18-lab5-browser-sqli-waf-403.png)

---

#### Khó khăn gặp phải & Cách giải quyết

| Vấn đề | Nguyên nhân | Cách giải quyết |
| :--- | :--- | :--- |
| WAF không chặn request SQLi khi gửi qua địa chỉ IP của EC2 | AWS WAF được gắn vào Application Load Balancer chứ không bảo vệ trực tiếp Public IP của EC2 | Điều chỉnh Security Group của EC2 chỉ cho phép nhận traffic HTTP port 80 từ Security Group của ALB, ép buộc mọi kết nối phải đi qua WAF |
| GuardDuty không hiển thị cảnh báo ngay lập tức sau khi quét Nmap | GuardDuty xử lý và phân tích log theo chu kỳ batch (thường mất từ 5 đến 15 phút để tổng hợp log) | Kiên nhẫn chờ từ 5-10 phút để GuardDuty thu thập đủ mẫu dữ liệu VPC Flow Logs và hiển thị cảnh báo trên console |
| Tệp tin VPC Flow Logs trong S3 không đọc được trực tiếp | Flow Logs tự động nén dưới định dạng Gzip (`.gz`) để tối ưu dung lượng lưu trữ | Tải tệp tin về máy tính cá nhân, sử dụng công cụ giải nén (như 7-Zip, WinRAR) hoặc lệnh `gunzip` trước khi đọc nội dung |
| Không gán được khóa KMS để mã hóa ổ đĩa EBS | Khóa Customer Managed Key (CMK) chưa cấp quyền sử dụng cho dịch vụ EC2 trong Key Policy | Cập nhật Key Policy của KMS cho phép service principal `ec2.amazonaws.com` thực hiện các action `kms:CreateGrant`, `kms:Decrypt`, `kms:GenerateDataKey` |
| Điểm đánh giá tuân thủ Security Hub thấp (30%) | Tài khoản mới thiếu các thiết lập bảo mật cấp tổ chức nâng cao (CloudTrail multi-region, MFA delete, S3 log auditing) | Rà soát danh sách 16 mục Failed trên Security Hub và thực hiện kích hoạt từng quy chuẩn theo khuyến nghị của CIS Benchmark |

### Kế hoạch tuần 6: Tự động hóa & CI/CD trên AWS

- Tìm hiểu giải pháp triển khai mã nguồn liên tục với bộ dịch vụ **AWS CodePipeline**, **AWS CodeBuild** và **AWS CodeDeploy**.
- Thiết lập quy trình tự động hóa kiểm thử mã nguồn và build Docker container image.
- Quản lý kho lưu trữ hình ảnh container an toàn với **Amazon Elastic Container Registry (ECR)**.
- Triển khai ứng dụng container hóa không cần quản lý máy chủ với **Amazon ECS (Elastic Container Service) on AWS Fargate**.
- Cấu hình thông báo tự động trạng thái build và pipeline thông qua **Amazon SNS** gửi về Email/Discord.
