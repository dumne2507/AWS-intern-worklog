---
title: "Cấu hình AWS IAM Role"
date: 2026-11-27
weight: 3
chapter: false
pre: "<b>5.3. </b>"
---

Hàm Lambda cần được cấp quyền đọc dữ liệu từ DynamoDB, gọi mô hình hội thoại trên Amazon Bedrock và truy vấn bản đồ từ Amazon Location Service (`geo-places` v2).

#### Các bước thực hiện

1. Truy cập **IAM → Roles → Create role**.
2. Chọn **AWS service** → **Lambda** → bấm **Next**.
3. Đặt tên Role là `RecipesLambdaRole` và bấm **Create role**.
4. Vào `RecipesLambdaRole`, chọn **Add permissions → Create inline policy → đặt tên là `RecipesLambdaPolicy`** và dán nội dung JSON:

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "ReadMealRecipesTable",
			"Effect": "Allow",
			"Action": [
				"dynamodb:Scan",
				"dynamodb:Query",
				"dynamodb:GetItem"
			],
			"Resource": "arn:aws:dynamodb:ap-southeast-1:*:table/RecipesTable"
		},
		{
			"Sid": "SearchNearbyStoresV2",
			"Effect": "Allow",
			"Action": [
				"geo-places:SearchText"
			],
			"Resource": "*"
		},
		{
			"Sid": "InvokeBedrockChefAI",
			"Effect": "Allow",
			"Action": [
				"bedrock:InvokeModel"
			],
			"Resource": [
				"arn:aws:bedrock:ap-southeast-1:*:inference-profile/apac.amazon.nova-micro-v1:0",
				"arn:aws:bedrock:*::foundation-model/amazon.nova-micro-v1:0"
			]
		},
		{
			"Sid": "WriteCloudWatchLogs",
			"Effect": "Allow",
			"Action": [
				"logs:CreateLogGroup",
				"logs:CreateLogStream",
				"logs:PutLogEvents"
			],
			"Resource": "arn:aws:logs:ap-southeast-1:*:log-group:/aws/lambda/RecipeSearchFunction:*"
		}
	]
}
```

5. Mở lại `RecipesLambdaRole`, chọn **Add permissions → Create inline policy → đặt tên là AmazonLocationGeoPlacesAccess** và dán nội dung JSON:

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"geo-places:SearchNearby",
				"geo-places:SearchText",
				"geo-places:Geocode",
				"geo-places:ReverseGeocode"
			],
			"Resource": "*"
		}
	]
}
```
