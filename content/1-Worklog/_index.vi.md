---
title: "Nhật ký công việc"
date: 2026-09-14
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

## Tổng quan Nhật ký Thực tập 12 Tuần

Trong suốt **12 tuần** tham gia chương trình thực tập **First Cloud AI Journey (FCAJ)** (bắt đầu từ **14/09/2026** đến đầu tháng **12/2026**), mình đã hoàn thành lộ trình đào tạo và thực hành toàn diện từ kiến thức nền tảng AWS đến thiết kế hệ thống Cloud/Serverless/Container và dự án cuối kỳ cùng nhóm (**Smart Kitchen / Bếp Thông Minh**).

### Cách thức hoàn thành Worklog

- **Lộ trình từng tuần (Thứ 2 – Thứ 7):** Các tuần được sắp xếp tăng dần từ dịch vụ lõi và mạng (Tuần 1–3), bảo mật và vận hành (Tuần 4–6), tự động hóa hạ tầng, Serverless và Container/CI-CD (Tuần 7–10), đến dịch chuyển hệ thống và phân tích dữ liệu (Tuần 11–12).
- **Thực hành trực tiếp trên AWS:** Phần lớn nội dung đều được triển khai thực tế trên tài khoản AWS (`us-east-1` / `ap-southeast-1`) thông qua AWS Management Console, AWS CloudShell, AWS CLI, `kubectl`, Docker, Python và CloudFormation/CDK, kèm ảnh chụp màn hình minh chứng kết quả và nhật ký xử lý lỗi.
- **Tổng hợp & Đối chiếu:** Cuối mỗi tuần đều ghi chép lại mục tiêu, các bước cấu hình trọng tâm, khó khăn gặp phải và cách khắc phục bằng cả Tiếng Việt lẫn Tiếng Anh.

### Danh mục nội dung 12 tuần

| Tuần | Ngày bắt đầu | Chủ đề chính | Nội dung trọng tâm |
| :---: | :---: | :--- | :--- |
| **1** | 14/09/2026 | [**Khởi tạo tài khoản & Dịch vụ cơ bản**](1.1-week1/) | Tạo tài khoản AWS Free Tier, Billing Alarm, EC2 cơ bản, EBS & Amazon S3 |
| **2** | 21/09/2026 | [**Mạng & CDN**](1.2-week2/) | Thiết kế VPC, Subnets, Route Tables, Internet/NAT Gateway, CloudFront & Route 53 |
| **3** | 28/09/2026 | [**Database & Scale**](1.3-week3/) | Amazon RDS Multi-AZ, Read Replicas, ElastiCache, Application Load Balancer & Auto Scaling |
| **4** | 05/10/2026 | [**Phân quyền & IAM**](1.4-week4/) | IAM Users/Roles/Policies, Permission Boundaries, STS & IAM Identity Center |
| **5** | 12/10/2026 | [**Bảo mật hệ thống**](1.5-week5/) | Security Groups vs. NACLs, mã hóa AWS KMS, AWS WAF, Secrets Manager & Shield |
| **6** | 19/10/2026 | [**Quản trị Vận hành**](1.6-week6/) | CloudWatch Metrics/Logs/Alarms, truy vết CloudTrail, AWS Config & Systems Manager |
| **7** | 26/10/2026 | [**IaC (Code hạ tầng)**](1.7-week7/) | Tự động hóa triển khai hạ tầng bằng AWS CloudFormation và AWS CDK |
| **8** | 02/11/2026 | [**Serverless**](1.8-week8/) | Xây dựng kiến trúc không máy chủ với AWS Lambda, API Gateway, DynamoDB & Step Functions |
| **9** | 09/11/2026 | [**Containers (Docker)**](1.9-week9/) | Đóng gói Docker & Amazon ECR, Lightsail Containers, ECS Fargate với ALB & Auto Scaling |
| **10** | 16/11/2026 | [**EKS & CI/CD**](1.10-week10/) | Cụm Amazon EKS & `kubectl`, CodeCommit, CodeBuild & CodePipeline triển khai lên ECS Fargate |
| **11** | 23/11/2026 | [**Dịch chuyển (Migration) & Kiến trúc Hướng sự kiện**](1.11-week11/) | AWS VM Import (`import-image`), AWS DMS đồng bộ CDC sang RDS, EventBridge + SQS + Lambda |
| **12** | 28/11/2026 | [**Data Analytics & Tổng kết chương trình**](1.12-week12/) | S3 Data Lake, Amazon Athena & Glue Data Catalog, Savings Plans/RI, Dọn tài nguyên & Hugo |
