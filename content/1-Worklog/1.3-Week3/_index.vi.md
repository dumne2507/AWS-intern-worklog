---
title: "Worklog Tuần 3"
date: 2026-09-28
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

## Mục tiêu tuần 3: Database & Scale

- **Amazon RDS:** Khởi tạo cơ sở dữ liệu quan hệ MySQL trên AWS RDS, cấu hình tham số bảo mật, kết nối an toàn từ máy chủ ảo EC2.
- **Amazon DynamoDB:** Thiết kế bảng NoSQL, tìm hiểu cơ chế Partition Key, phân tích sự khác biệt về hiệu năng và chi phí giữa Scan và Query.
- **ElastiCache:** Thiết lập cụm bộ nhớ đệm Redis In-memory với tính năng Multi-AZ và Auto-failover để tối ưu tốc độ đọc và giảm tải cho Database.
- **Auto Scaling:** Xây dựng Launch Template định nghĩa cấu hình máy chủ chuẩn và cấu hình EC2 Auto Scaling Group tự động duy trì số lượng máy chủ mong muốn.
- **Load Balancer:** Thiết lập Application Load Balancer (ALB) phân phối lưu lượng truy cập cân bằng giữa các máy chủ Web trên nhiều Availability Zone (Multi-AZ).
- **Lab Tổng hợp 3:** Xây dựng kiến trúc High Availability hoàn chỉnh (ALB + Auto Scaling EC2 + RDS), kiểm chứng cơ chế tự phục hồi (Self-Healing / Auto-Recovery).

### Các công việc cần triển khai trong tuần này

| Thứ | Chủ đề | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **2** | **Amazon RDS** | Khởi tạo MySQL trên RDS, cấu hình endpoint bảo mật, kết nối từ EC2 qua MariaDB client | 28/09/2026 | 28/09/2026 | [Amazon RDS](https://docs.aws.amazon.com/rds/) |
| **3** | **Amazon DynamoDB** | Tạo bảng NoSQL (`LabT3_Products`), thiết lập Partition Key, thực hành truy vấn Scan vs Query | 29/09/2026 | 29/09/2026 | [Amazon DynamoDB](https://docs.aws.amazon.com/dynamodb/) |
| **4** | **ElastiCache** | Thiết lập cụm Redis Cache (`labt4-rediscache`), kích hoạt Multi-AZ, Auto-failover và mã hóa dữ liệu | 30/09/2026 | 30/09/2026 | [Amazon ElastiCache](https://docs.aws.amazon.com/elasticache/) |
| **5** | **Auto Scaling** | Khởi tạo Launch Template (`LabT5-AutoScaling`), cấu hình EC2 Auto Scaling Group tự động duy trì máy chủ | 01/10/2026 | 01/10/2026 | [EC2 Auto Scaling](https://docs.aws.amazon.com/autoscaling/) |
| **6** | **Load Balancer** | Cấu hình Application Load Balancer (`Web-LB`), tạo Target Group và kiểm tra cân bằng tải giữa các AZ | 02/10/2026 | 02/10/2026 | [Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/) |
| **7** | **Lab Tổng hợp 3** | Dựng kiến trúc High Availability: ALB + Auto Scaling EC2 + Database, kiểm chứng Self-Healing | 03/10/2026 | 03/10/2026 | [AWS Architecture Center](https://aws.amazon.com/architecture/) |

### Kết quả đạt được tuần 3

- Khởi tạo thành công cơ sở dữ liệu quan hệ Amazon RDS MySQL (`labt2-db`), kết nối an toàn từ máy chủ EC2 (`MyLabT2`) qua client MariaDB.
- Thiết kế bảng NoSQL Amazon DynamoDB (`LabT3_Products`) với Partition Key `ProductID`, chứng minh phương thức Query chỉ tiêu tốn 0.5 RCU so với 2 RCUs của Scan.
- Thiết lập cụm bộ nhớ đệm Amazon ElastiCache for Redis (`labt4-rediscache`) 2 node đạt chuẩn High Availability với Multi-AZ, Auto-failover và mã hóa In-transit & At-rest.
- Tự động hóa triển khai máy chủ với Launch Template `LabT5-AutoScaling`, khởi chạy tự động EC2 instance thông qua Auto Scaling Group.
- Triển khai thành công Application Load Balancer (`Web-LB`) điều phối lưu lượng truy cập mượt mà theo thuật toán Round-Robin giữa Web Server Khu vực A và Khu vực B.
- Hoàn thiện mô hình High Availability toàn diện (ALB + Auto Scaling EC2 + RDS), kiểm chứng tính năng Self-Healing: khi 1 instance bị lỗi và terminate, Auto Scaling Group lập tức tự động khởi chạy instance thay thế để duy trì hệ thống luôn sẵn sàng 100%.

---

#### Bài tập 1: Amazon RDS — Khởi tạo MySQL trên RDS & Kết nối từ EC2

Khởi tạo cơ sở dữ liệu quan hệ **Amazon RDS** chạy engine MySQL mang tên **`labt2-db`** tại Region `us-east-1`. Hệ thống cấp phát Endpoint kết nối chuyên dụng:
- **Endpoint:** `labt2-db.c0pysocmgc4s.us-east-1.rds.amazonaws.com`
- **Port:** `3306`
- **Master username:** `admin`
- **IAM Authentication:** Disabled | **Internet access gateway:** Disabled (bảo mật nội bộ)

AWS RDS cung cấp hướng dẫn tải chứng chỉ bảo mật SSL/TLS `global-bundle.pem` để mã hóa kết nối giữa máy chủ ứng dụng và cơ sở dữ liệu.

> **Screenshot:** ![Thông tin Endpoint và cấu hình kết nối Amazon RDS MySQL](/images/1-Worklog/week-03/01-rds-database-connect-info.png)

Kết nối vào máy chủ EC2 **`MyLabT2`** (`i-0d40f82022fa610a8`, Public IPv4: `54.87.188.14`, Private IPv4: `172.31.43.186`). Tiến hành cài đặt client MySQL/MariaDB và thực hiện kết nối:

```bash
# Cài đặt MariaDB client trên Amazon Linux 2023
sudo dnf install mariadb105 -y

# Kết nối tới Amazon RDS MySQL
mysql -h labt2-db.c0pysocmgc4s.us-east-1.rds.amazonaws.com -P 3306 -u admin -p

# Nhập mật khẩu quản trị viên
# Kết quả kết nối thành công:
# Welcome to the MariaDB monitor. Commands end with ; or \g.
# Your MySQL connection id is 35
# Server version: 8.4.9 Source distribution
# MySQL [(none)]>
```

Quá trình kết nối thành công xác minh rằng Security Group của RDS đã mở Inbound port 3306 đúng dải mạng hoặc Security Group của máy chủ EC2.

> **Screenshot:** ![EC2 kết nối thành công vào cơ sở dữ liệu Amazon RDS](/images/1-Worklog/week-03/02-ec2-rds-mysql-connected.png)

---

#### Bài tập 2: Amazon DynamoDB — Tạo bảng NoSQL, Partition Key & Thực hành truy vấn

Khởi tạo bảng NoSQL **`LabT3_Products`** trên dịch vụ **Amazon DynamoDB**. Cấu hình **Partition Key** là trường **`ProductID`** (dạng String) làm khóa chính định danh duy nhất cho từng bản ghi. Thêm các sản phẩm mẫu với các trường dữ liệu linh hoạt (schemaless):
- Sản phẩm 1: `ProductID = SP01`, `ProductName = Laptop Dell`, `Price = 15000000`
- Sản phẩm 2: `ProductID = SP02`, `ProductName = Logitech mouse`, `Color = Black`

Thực hiện thao tác **Scan**: Hệ thống phải duyệt qua toàn bộ dữ liệu trong bảng, quét 2 items, trả về 2 items và tiêu thụ **2 RCUs** (Read Capacity Units).

> **Screenshot:** ![Thao tác Scan trên bảng DynamoDB tiêu thụ 2 RCUs](/images/1-Worklog/week-03/03-dynamodb-scan-products.png)

Thực hiện thao tác **Query**: Lọc trực tiếp bằng Partition Key `ProductID = SP01`. Hệ thống truy xuất trực tiếp vị trí lưu trữ bản ghi dựa vào hàm băm của khóa chính, chỉ quét đúng 1 item và trả về 1 item, tiêu thụ chỉ **0.5 RCU** (tiết kiệm 75% chi phí đọc so với Scan).

> **Screenshot:** ![Thao tác Query bằng Partition Key tối ưu chi phí 0.5 RCU](/images/1-Worklog/week-03/04-dynamodb-query-partition-key.png)

Kết quả truy vấn Query hiển thị chi tiết thông tin sản phẩm `SP01` chính xác và tức thì với hiệu năng vượt trội.

> **Screenshot:** ![Kết quả truy vấn DynamoDB chi tiết của sản phẩm SP01](/images/1-Worklog/week-03/05-dynamodb-query-result.png)

---

#### Bài tập 3: ElastiCache — Thiết lập Cụm Redis Cache Giảm tải cho Database

Khởi tạo cụm bộ nhớ đệm In-memory **Amazon ElastiCache for Redis** mang tên **`labt4-rediscache`** với cấu hình chuẩn doanh nghiệp:
- **Engine:** Redis version `7.1.0`
- **Node type:** `cache.t3.micro`
- **Number of nodes:** `2` (Gồm 1 Primary Node và 1 Read Replica Node)
- **Multi-AZ:** Enabled (Đảm bảo tính sẵn sàng cao phân tán trên nhiều AZ)
- **Auto-failover:** Enabled (Tự động chuyển đổi dự phòng khi Primary node gặp sự cố)
- **Encryption:** Encryption at rest (Enabled), Encryption in transit (Required)
- **Primary endpoint:** `master.labt4-rediscache.rfu6sh.use1.cache.amazonaws.com:6379`
- **Reader endpoint:** `replica.labt4-rediscache.rfu6sh.use1.cache.amazonaws.com:6379`
- **Trạng thái:** ✅ Available

Cụm ElastiCache cho phép ứng dụng đọc dữ liệu cache siêu tốc với độ trễ dưới 1 mili-giây, giảm tải triệt để các câu lệnh SELECT nặng nề tới Amazon RDS.

> **Screenshot:** ![Cụm ElastiCache Redis với Multi-AZ và Auto-failover](/images/1-Worklog/week-03/06-elasticache-redis-cluster.png)

---

#### Bài tập 4: Auto Scaling — Khởi tạo Launch Template & Cấu hình EC2 Auto Scaling Group

Tạo **Launch Template** mang tên **`LabT5-AutoScaling`** (Template ID: `lt-0034af3e5980ae08d`, Version 1). Launch Template lưu trữ toàn bộ cấu hình máy chủ chuẩn:
- AMI: Amazon Linux 2023
- Instance type: `t2.micro`
- Network settings: Gắn Security Group cho phép HTTP port 80
- User Data script: Tự động cài đặt web server Apache khi instance khởi động

> **Screenshot:** ![Launch Template LabT5-AutoScaling đã tạo thành công](/images/1-Worklog/week-03/07-launch-template-created.png)

Tạo **Auto Scaling Group** dựa trên Launch Template vừa tạo, đặt Desired Capacity = 1, Minimum = 1, Maximum = 3. Auto Scaling Group tự động tính toán và khởi chạy instance **`i-05e9debdebe0517`** tại Availability Zone `us-east-1d`. Instance chuyển sang trạng thái **Running** và tự động hoàn thành kiểm tra sức khỏe ban đầu.

> **Screenshot:** ![Instance EC2 được Auto Scaling Group tự động khởi chạy](/images/1-Worklog/week-03/08-asg-instance-running.png)

---

#### Bài tập 5: Load Balancer — Cấu hình Application Load Balancer (ALB) Phân tải Traffic

Khởi tạo **Application Load Balancer (ALB)** mang tên **`Web-LB`** với các thông số:
- **Loại:** Application Load Balancer (Layer 7 HTTP/HTTPS)
- **Scheme:** Internet-facing (Đón nhận traffic trực tiếp từ người dùng Internet)
- **VPC:** `vpc-002805dad0cf2c1ee`
- **Availability Zones:** Bao phủ 2 AZ độc lập gồm `subnet-06775fc833935f53a` (`us-east-1a`) và `subnet-0213687910a63f131` (`us-east-1b`)
- **DNS Name:** `Web-LB-136874904.us-east-1.elb.amazonaws.com` (A Record)

> **Screenshot:** ![Chi tiết khởi tạo Application Load Balancer Web-LB](/images/1-Worklog/week-03/09-alb-provisioning-details.png)

Cấu hình Target Group gồm 2 máy chủ Web Server đặt tại 2 vùng Availability Zone khác nhau. Khi truy cập vào đường dẫn DNS của Load Balancer:
- Lần truy cập thứ nhất: Load Balancer phân phối request tới **Máy chủ WEB 1 - Khu vực A**.

> **Screenshot:** ![Load Balancer điều phối request tới Web Server 1 Khu vực A](/images/1-Worklog/week-03/10-alb-target-web1-response.png)

- Lần truy cập thứ hai (Refresh): Load Balancer luân chuyển theo cơ chế Round-Robin sang **Máy chủ WEB 2 - Khu vực B**.

> **Screenshot:** ![Load Balancer luân chuyển request sang Web Server 2 Khu vực B](/images/1-Worklog/week-03/11-alb-target-web2-response.png)

---

#### Bài tập 6: Lab Tổng hợp 3 — Kiến trúc High Availability: ALB + Auto Scaling EC2 + Database

Xây dựng hệ thống hoàn chỉnh kết hợp **Application Load Balancer**, **Auto Scaling Group** và **Cơ sở dữ liệu Multi-AZ**. Auto Scaling Group duy trì 2 instances hoạt động song song tại 2 Availability Zone khác nhau:
- `i-060adb40952d798d8` tại `us-east-1a` (Running, 2/2 checks passed)
- `i-05fd991ecb5911ea0` tại `us-east-1b` (Running, 2/2 checks passed)

> **Screenshot:** ![Hệ thống Auto Scaling Group chạy trên Multi-AZ](/images/1-Worklog/week-03/12-ha-asg-instances-multi-az.png)

Kiểm tra truy cập thông qua DNS của ALB, website phản hồi mượt mà với thông điệp: *"Hello from Auto Scaling Group"*.

> **Screenshot:** ![Truy cập web thành công qua Application Load Balancer](/images/1-Worklog/week-03/13-ha-alb-asg-web-success.png)

**Kiểm chứng cơ chế Tự phục hồi (Self-Healing / Auto-Recovery):**
Tiến hành kiểm thử độ bền hệ thống bằng cách chủ động dừng/xóa instance `i-012375caa815c8465` (`Web-ALB`). Health check của ALB ngay lập tức phát hiện instance bị unhealthy, và Auto Scaling Group tự động khởi chạy tức thời một instance mới `i-059fb2b8e7c73f56f` để bù đắp, đảm bảo số lượng máy chủ mong muốn luôn bằng 2.

> **Screenshot:** ![Auto Scaling Group tự động khởi chạy instance mới bù đắp](/images/1-Worklog/week-03/14-ha-asg-auto-healing-recovery.png)

Sau khi hoàn tất quá trình kiểm thử kiến trúc High Availability, tiến hành dọn dẹp các tài nguyên (Terminate instances, xóa ASG và Load Balancer) để tránh phát sinh chi phí ngoài ý muốn trên tài khoản AWS Free Tier.

> **Screenshot:** ![Dọn dẹp và kết thúc các máy chủ thử nghiệm](/images/1-Worklog/week-03/15-cleanup-terminated-instances.png)

---

#### Khó khăn gặp phải & Cách giải quyết

| Vấn đề | Nguyên nhân | Cách giải quyết |
| :--- | :--- | :--- |
| Kết nối từ EC2 tới Amazon RDS MySQL bị timeout | Security Group của RDS chưa mở Inbound port 3306 cho EC2 | Thêm Inbound Rule cho RDS Security Group: Type MySQL/Aurora (port 3306), Source trỏ về Security Group của máy chủ EC2 |
| DynamoDB Scan tiêu tốn nhiều RCU và chi phí đọc | Lệnh Scan duyệt toàn bộ bảng thay vì truy cập theo chỉ mục | Sử dụng thao tác **Query** với Partition Key cụ thể (`ProductID = SP01`) giúp giảm 75% chi phí đọc (chỉ tốn 0.5 RCU) |
| Không kết nối được ElastiCache Redis từ máy ngoài | Cụm Redis đặt trong private subnet và bật Transit Encryption (TLS) | Kết nối thông qua máy chủ EC2 trong cùng VPC và sử dụng cờ `--tls` khi chạy `redis-cli` |
| Load Balancer báo Target unhealthy (Health Check Failed) | Web server trên EC2 chưa khởi động hoặc chưa mở port 80 trong Security Group | Kiểm tra trạng thái service Apache (`sudo systemctl status httpd`) và mở Inbound port 80 cho Security Group của EC2 từ ALB |
| Instance thay thế trong Auto Scaling không tự động chạy web server | Launch Template chưa gắn User Data script cài đặt web | Bổ sung script `yum install -y httpd` trong phần User Data của Launch Template để tự động hóa hoàn toàn |

### Kế hoạch tuần 4: Serverless & Application Integration

- Tìm hiểu kiến trúc Serverless với AWS Lambda và Amazon API Gateway.
- Xây dựng mô hình kiến trúc hướng sự kiện (Event-Driven Architecture) với Amazon SNS và Amazon SQS.
- Thiết lập quy trình xử lý bất đồng bộ (Asynchronous Decoupled Processing).
- Tìm hiểu Amazon EventBridge và kết hợp các dịch vụ Serverless.
