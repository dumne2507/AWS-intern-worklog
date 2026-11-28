---
title: "Dọn dẹp tài nguyên"
date: 2026-11-27
weight: 8
chapter: false
pre: "<b>5.8. </b>"
---

Để tránh phát sinh chi phí sau khi hoàn tất bài thực hành, hãy xóa các tài nguyên đã tạo trong `ap-southeast-1` (và các dịch vụ toàn cầu) theo đúng thứ tự sau:

1. **Amazon CloudFront:** Tắt (**Disable**) và xóa Distribution trỏ tới `recipe-suggestion-website`.
2. **Amazon S3:** Làm rỗng (**Empty**) và xóa bucket `recipe-suggestion-website`.
3. **Amazon API Gateway:** Xóa HTTP API `BepThongMinhAPI`.
4. **AWS Lambda:** Xóa hàm `RecipeSearchFunction`.
5. **Amazon CloudWatch Logs:** Xóa log group `/aws/lambda/RecipeSearchFunction`.
6. **Amazon DynamoDB:** Xóa bảng `RecipesTable`.
7. **AWS IAM:** Xóa role `RecipesLambdaRole` (kèm các inline policy `RecipesLambdaPolicy` và `AmazonLocationGeoPlacesAccess`).

> **Lưu ý:** **Amazon Bedrock** (`apac.amazon.nova-micro-v1:0`) và **Amazon Location Service** (`geo-places` v2) hoạt động theo cơ chế serverless pay-per-request nên không có tài nguyên cố định cần xóa.