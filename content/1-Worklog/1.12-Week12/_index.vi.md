---
title: "Worklog Tuần 12"
date: 2026-11-28
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

## Mục tiêu tuần 12: Data Analytics & Tổng kết chương trình

- **Kiến trúc AWS Data Lake:** Hiểu và thiết kế kiến trúc **Data Lake** tập trung trên **Amazon S3**, áp dụng cơ chế phân lớp lưu trữ, phân vùng (Partitioning), mã hóa và phân quyền chi tiết cho dữ liệu thô lẫn dữ liệu đã chuẩn hóa.
- **Truy vấn dữ liệu Serverless:** Sử dụng **Amazon Athena** kết hợp **AWS Glue Data Catalog** để truy vấn trực tiếp dữ liệu phi cấu trúc và bán cấu trúc (`.csv`, `.json`, `.parquet`) trên S3 bằng chuẩn SQL không cần quản trị máy chủ.
- **Quản lý chi phí nâng cao:** Nắm vững chiến lược tối ưu hóa ngân sách đám mây dài hạn thông qua **Savings Plans** và **Reserved Instances (RI)**, kết hợp **AWS Cost Explorer** và **AWS Budgets**.
- **Tổng hợp kiến trúc & Dọn dẹp tài nguyên:** Hệ thống hóa toàn bộ kiến trúc AWS đã triển khai xuyên suốt 12 tuần thực tập, rà soát và dọn dẹp triệt để tài nguyên nhằm tránh phát sinh chi phí ngoài ý muốn (**Cost Hygiene**).
- **Hoàn thiện Hugo & Xuất bản GitHub Pages:** Chuẩn hóa cấu trúc source code **Hugo**, kiểm tra tính nhất quán song ngữ, đồng bộ lên **GitHub** và kích hoạt pipeline **GitHub Actions / GitHub Pages** xuất bản website Worklog chính thức.

---

### Các công việc triển khai trong tuần

| Thứ | Chủ đề | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **2** | **AWS Data Lake trên S3** | Thiết kế Data Lake đa vùng (`raw-zone/`, `processed-zone/`), Partitioning, mã hóa & phân quyền IAM | 30/11/2026 | 30/11/2026 | [Data Lakes on AWS](https://docs.aws.amazon.com/whitepapers/latest/building-data-lakes/building-data-lake-aws.html) |
| **3** | **Truy vấn với Amazon Athena** | Tìm hiểu AWS Glue Data Catalog / Crawler, cấu hình Query Result Location, tối ưu SQL & Parquet | 01/12/2026 | 01/12/2026 | [Amazon Athena User Guide](https://docs.aws.amazon.com/athena/latest/ug/) |
| **4** | **Quản lý Chi phí Nâng cao** | So sánh Savings Plans và Reserved Instances (RI), ứng dụng Cost Explorer & AWS Budgets | 02/12/2026 | 02/12/2026 | [AWS Savings Plans](https://docs.aws.amazon.com/savingsplans/latest/userguide/) |
| **5** | **Đánh giá & Dọn dẹp tài nguyên** | Hệ thống hóa kiến trúc 12 tuần, kiểm tra Billing và xóa sạch tài nguyên tránh phát sinh phí | 03/12/2026 | 03/12/2026 | [AWS Billing & Cost Management](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/) |
| **6** | **Chuẩn hóa Hugo & Xuất bản GitHub** | Rà soát nội dung 12 tuần song ngữ, chuẩn hóa thư mục Hugo, kiểm tra `hugo.yml` & xuất bản GitHub Pages | 04/12/2026 | 04/12/2026 | [Hugo Documentation](https://gohugo.io/documentation/) |

### Kết quả đạt được tuần 12

- Thiết kế hoàn chỉnh kiến trúc **S3 Data Lake** tách biệt rõ các vùng `raw-zone/`, `processed-zone/` và `athena-results/`, áp dụng phân vùng theo thời gian (`year=/month=/day=`) và mã hóa chuẩn SSE-S3/SSE-KMS.
- Nắm vững quy trình phân tích dữ liệu phi máy chủ kết hợp **AWS Glue Crawler**, **Glue Data Catalog** và **Amazon Athena** (Presto/Trino), cùng các kỹ thuật cắt giảm 70–90% dung lượng dữ liệu quét (chuyển sang Parquet, lọc Partition, tránh `SELECT *`).
- Phân tích rõ sự khác biệt giữa **Compute Savings Plans**, **EC2 Instance Savings Plans** và **Standard/Convertible Reserved Instances** nhằm đạt mức tiết kiệm tới **66–72%** so với On-Demand cho hệ thống Production dài hạn.
- Hoàn tất quy trình rà soát hóa đơn, dọn dẹp tài nguyên thực hành (**Cost Hygiene**) và đồng bộ hóa toàn bộ báo cáo thực tập song ngữ lên **GitHub Pages**.

---

### Nội dung chi tiết theo ngày

#### Thứ 2: Xây dựng AWS Data Lake với Amazon S3

- **Khái niệm cốt lõi:** Data Lake cho phép lưu trữ tập trung toàn bộ dữ liệu thô (Raw data) ở quy mô lớn mà không cần định nghĩa trước lược đồ (**Schema-on-Read**), phục vụ linh hoạt cho phân tích BI và Machine Learning.
- **Kiến trúc phân lớp lưu trữ (Storage Tiers / Zones) trên S3:**
  - `raw-zone/` (**Landing Zone**): Lưu trữ dữ liệu nguyên bản đổ về từ các nguồn (Log hệ thống, IoT, Database export) dưới định dạng `.csv` hoặc `.json`.
  - `processed-zone/` (**Curated Zone**): Lưu trữ dữ liệu đã được làm sạch, chuẩn hóa và chuyển đổi sang định dạng tối ưu hiệu năng truy vấn dạng cột như **Apache Parquet** hoặc **ORC**.
  - `athena-results/`: Bucket hoặc prefix chuyên biệt lưu trữ tệp kết quả truy vấn (Query Results) và metadata của Amazon Athena.
- **Cơ chế tối ưu & Bảo mật:**
  - **Partitioning (Phân vùng dữ liệu):** Tổ chức thư mục theo cấu trúc phân cấp thời gian (ví dụ: `s3://datalake-bucket/raw/year=2026/month=11/day=28/`) giúp giảm thiểu lượng dữ liệu quét (**Data Scanned**) và tăng tốc độ truy vấn.
  - **Phân quyền truy cập:** Áp dụng nguyên tắc đặc quyền tối thiểu (**Least Privilege**) thông qua **IAM Policies** và **S3 Bucket Policies**, kết hợp **Block Public Access** và mã hóa mặc định **SSE-S3 / SSE-KMS**.

---

#### Thứ 3: Truy vấn dữ liệu Serverless với Amazon Athena

- **Kiến trúc hoạt động:** **Amazon Athena** là dịch vụ truy vấn tương tác dựa trên engine mã nguồn mở **Presto / Trino**, cho phép chạy câu lệnh SQL trực tiếp trên S3 mà không cần khởi tạo hay quản trị cụm máy chủ.
- **Quy trình triển khai chuẩn:**
  1. **Tạo Metadata với AWS Glue Data Catalog:** Sử dụng **AWS Glue Crawler** quét tự động thư mục trên S3 để suy luận cấu trúc cột (Schema) và tạo bảng định nghĩa trong Glue Database, hoặc định nghĩa thủ công bằng lệnh DDL `CREATE EXTERNAL TABLE`.
  2. **Cấu hình Query Result Location:** Chỉ định đường dẫn S3 lưu tệp xuất kết quả (`.csv` và `.metadata`) trong Workgroup trước khi thực thi truy vấn.
  3. **Thực thi truy vấn SQL & Tối ưu chi phí:**
     - Athena tính phí dựa trên dung lượng dữ liệu quét trên mỗi truy vấn (**\$5 / 1 TB dữ liệu quét**).
     - **Kỹ thuật tối ưu:** Sử dụng mệnh đề `WHERE` kết hợp với các cột Partition (`year`, `month`, `day`), tránh dùng `SELECT *` (chỉ chọn các cột cần thiết), nén tệp và chuyển đổi sang định dạng lưu trữ dạng cột (**Apache Parquet**) để giảm tới **70–90%** dung lượng quét.

---

#### Thứ 4: Quản lý Chi phí Nâng cao (Savings Plans & Reserved Instances)

- **Bài toán tối ưu ngân sách dài hạn:** Mô hình **On-Demand** mang lại sự linh hoạt cao nhưng chi phí đơn vị lớn đối với các hệ thống chạy ổn định 24/7 (**Production**). AWS cung cấp hai cơ chế cam kết sử dụng 1 năm hoặc 3 năm để giảm tới **72% chi phí** so với On-Demand.
- **So sánh chuyên sâu Savings Plans và Reserved Instances (RI):**

| Tiêu chí | Savings Plans | Reserved Instances (RI) |
| :--- | :--- | :--- |
| **Đơn vị cam kết** | Cam kết mức chi tiêu theo giờ (**\$/giờ**) trong 1 hoặc 3 năm. | Cam kết cấu hình tài nguyên cụ thể (Instance type, Region, OS) trong 1 hoặc 3 năm. |
| **Phạm vi áp dụng** | **Compute Savings Plans:** Áp dụng linh hoạt cho cả EC2, AWS Fargate và AWS Lambda (không phân biệt Region, OS, dòng máy).<br>**EC2 Instance Savings Plans:** Áp dụng cho một họ Instance (ví dụ: `t3`, `m5`) trong một Region cụ thể. | Chỉ áp dụng cho đúng dòng máy và Region đã mua (**Standard RI**) hoặc cho phép đổi dòng máy tương đương (**Convertible RI**). |
| **Mức tiết kiệm tối đa** | Lên tới **66%** (Compute) và **72%** (EC2 Instance). | Lên tới **72%** (Standard RI). |
| **Độ linh hoạt** | **Rất cao** — tự động áp dụng mức giảm giá khi hệ thống chuyển đổi từ EC2 sang Container (ECS/Fargate) hoặc Serverless (Lambda). | **Thấp hơn** — phù hợp với các dịch vụ có kiến trúc cố định như Amazon RDS, ElastiCache, OpenSearch. |

- **Công cụ hỗ trợ ra quyết định:** Sử dụng **AWS Cost Explorer** (mục *Recommendations*) để phân tích lịch sử tiêu thụ 30–60 ngày gần nhất và đề xuất gói cam kết tối ưu, kết hợp **AWS Budgets** để thiết lập cảnh báo ngưỡng chi tiêu tự động qua email/SNS.

---

#### Thứ 5: Project Assessment & Dọn dẹp tài nguyên (Resource Cleanup)

- **Hệ thống hóa kiến trúc tổng thể 12 tuần:**
  - **Nền tảng mạng & Bảo mật:** Thiết kế VPC đa vùng sẵn sàng (**Multi-AZ**), Public/Private Subnets, NAT Gateway, Security Groups, NACLs; quản lý danh tính qua IAM Identity Center, Permission Boundaries và giám sát bằng CloudTrail / KMS.
  - **Compute, Containers & CI/CD:** Triển khai máy ảo EC2 Auto Scaling sau Application Load Balancer (ALB), đóng gói ứng dụng lên ECS Fargate, Lightsail Containers, Amazon EKS và tự động hóa CI/CD qua CodeCommit, CodeBuild, CodePipeline.
  - **Database & Migration:** Quản trị Multi-AZ RDS, thực hiện di dời máy ảo Lift-and-Shift (`vmimport`) và nhân bản dữ liệu trực tuyến/ngoại tuyến (Full Load + CDC) từ EC2 sang RDS bằng **AWS DMS**.
  - **Serverless, GenAI & Event-Driven:** Tái cấu trúc ứng dụng nguyên khối sang Microservices bất đồng bộ sử dụng API Gateway, Lambda, DynamoDB, EventBridge và SQS, đồng thời tích hợp AI tạo sinh qua Amazon Bedrock và bản đồ Amazon Location Service.
- **Quy trình rà soát và dọn dẹp tài nguyên tránh phát sinh phí (Cost Hygiene):**
  1. Kiểm tra mục **Bills** và **Cost Explorer** trên AWS Billing Console để xác định các Region đang phát sinh chi phí.
  2. Xóa các tài nguyên tính phí theo giờ trước: Dừng/xóa **DMS Replication Instances & Tasks**, xóa **RDS Instances** (bỏ chọn *Create final snapshot*), hủy **ECS/EKS Services**, xóa **NAT Gateways** và giải phóng **Elastic IPs** không sử dụng.
  3. Terminate toàn bộ **EC2 Instances**, xóa **Load Balancers (ALB)**, **EBS Volumes** và **Snapshots** còn sót lại.
  4. Dọn dẹp **S3 Buckets** (Empty bucket trước khi Delete), xóa **DynamoDB Tables**, **SQS Queues**, **EventBridge Rules**, **CloudFront Distributions** và **Lambda Functions**.

---

#### Thứ 6: Wrap-up & Chuẩn hóa cấu trúc Hugo trên GitHub

- **Tổng hợp nội dung 12 tuần:** Rà soát tính nhất quán về định dạng Markdown song ngữ (`_index.md` và `_index.vi.md`), kiểm tra hình ảnh sơ đồ kiến trúc, kết quả thực hành và bài học kinh nghiệm từ Tuần 1 đến Tuần 12.
- **Cấu trúc chuẩn thư mục website Hugo (Theme `hugo-theme-learn`):**
  ```text
  ├── hugo.toml                        # Cấu hình baseURL, theme, menu đa ngôn ngữ và shortcuts
  ├── content/
  │   ├── _index.md / _index.vi.md     # Trang chủ giới thiệu tổng quan hành trình thực tập (EN / VI)
  │   ├── 1-Worklog/
  │   │   ├── _index.md / _index.vi.md # Mục lục tổng hợp 12 tuần
  │   │   └── 1.1-Week1/ ... 1.12-Week12/ # Nội dung chi tiết từng tuần kèm ảnh minh họa
  │   └── 2-Proposal/ ... 7-Feedback/  # Đề xuất dự án, Blog, Sự kiện, Workshop, Tự đánh giá & Góp ý
  ├── static/
  │   └── images/                      # Toàn bộ hình ảnh minh họa lab được phân thư mục theo từng mục/tuần
  └── .github/
      └── workflows/
          └── hugo.yml                 # Pipeline GitHub Actions tự động build và deploy lên GitHub Pages
  ```
