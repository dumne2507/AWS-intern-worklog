---
title: "Workshop"
date: 2026-11-27
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Xây dựng Bếp Thông Minh: Workshop Ứng dụng Gợi ý Món ăn Serverless & AI Chef trên AWS

#### Tổng quan

Trong bài thực hành (Workshop) này, bạn sẽ xây dựng và triển khai hoàn chỉnh **Bếp Thông Minh (Smart Kitchen)** — ứng dụng web kiến trúc phi máy chủ (Serverless) trên AWS hỗ trợ gợi ý món ăn theo nguyên liệu/ngân sách, tư vấn ẩm thực bằng AI tạo sinh (Generative AI) và tìm kiếm địa điểm mua nguyên liệu gần vị trí người dùng.

Hệ thống tách biệt hoàn toàn tầng giao diện tĩnh và tầng xử lý nghiệp vụ nhằm tối ưu chi phí theo mô hình pay-as-you-go, tự động mở rộng và đảm bảo bảo mật HTTPS:

+ **Amazon S3 & Amazon CloudFront** — Lưu trữ giao diện tĩnh (`template.html`) và phân phối qua HTTPS nhằm hỗ trợ API định vị GPS của trình duyệt (`navigator.geolocation`)
+ **Amazon API Gateway (HTTP API)** — Cung cấp các endpoint HTTP (`GET /search`, `GET /nearby-stores`, `POST /chat`) kèm cấu hình CORS
+ **AWS Lambda (Python 3.12)** — Hàm xử lý tập trung (`RecipeSearchFunction`) thực hiện lọc công thức, gọi AI và truy vấn vị trí cửa hàng
+ **Amazon DynamoDB** — Lưu trữ cơ sở dữ liệu công thức nấu ăn (`RecipesTable`) theo chế độ On-Demand
+ **Amazon Bedrock** — Vận hành Chatbot Đầu bếp AI tâm lý với mô hình **Amazon Nova Micro** (`apac.amazon.nova-micro-v1:0`)
+ **Amazon Location Service (`geo-places` v2)** — Geocode địa chỉ nhập tay và tìm kiếm siêu thị/chợ gần nhất theo khoảng cách và mức ngân sách
+ **AWS IAM & Amazon CloudWatch** — Phân quyền tối thiểu qua Execution Role (`RecipesLambdaRole`) và giám sát log tập trung

#### Nội dung

1. [Tổng quan & Chuẩn bị](5.01-workshop-overview/)
2. [Tạo cơ sở dữ liệu DynamoDB](5.02-setup-dynamodb/)
3. [Cấu hình AWS IAM Role](5.03-configure-iam-role/)
4. [Triển khai AWS Lambda Backend](5.04-deploy-lambda-backend/)
5. [Thiết lập Amazon API Gateway](5.05-configure-api-gateway/)
6. [Triển khai Frontend lên S3 & CloudFront](5.06-deploy-s3-cloudfront/)
7. [Kiểm thử ứng dụng](5.07-test-application/)
8. [Dọn dẹp tài nguyên](5.08-cleanup/)
