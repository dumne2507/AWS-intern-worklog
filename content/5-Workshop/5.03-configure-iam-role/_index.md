---
title: "Configure AWS IAM Role"
date: 2026-11-27
weight: 3
chapter: false
pre: "<b>5.3. </b>"
---

The Lambda function requires permissions to read data from DynamoDB, invoke conversational models on Amazon Bedrock, and query maps from Amazon Location Service (`geo-places` v2).

#### Implementation Steps

1. Navigate to **IAM → Roles → Create role**.
2. Select **AWS service** → **Lambda** → click **Next**.
3. Name the role `RecipesLambdaRole` and click **Create role**.
4. Open `RecipesLambdaRole`, select **Add permissions → Create inline policy → name it RecipesLambdaPolicy**, and paste the following JSON policy:

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
5. Reopen `RecipesLambdaRole`, select **Add permissions → Create inline policy → name it `AmazonLocationGeoPlacesAccess`**, and paste the following JSON policy:

```JSON
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
