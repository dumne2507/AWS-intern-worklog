---
title: "Resource Cleanup"
date: 2026-11-27
weight: 8
chapter: false
pre: "<b>5.8. </b>"
---

To avoid incurring ongoing charges after completing the workshop, delete the created resources in `ap-southeast-1` (and global services) in the following order:

1. **Amazon CloudFront:** Disable and delete the CloudFront distribution pointing to `recipe-suggestion-website`.
2. **Amazon S3:** Empty and delete the `recipe-suggestion-website` bucket.
3. **Amazon API Gateway:** Delete the HTTP API `BepThongMinhAPI`.
4. **AWS Lambda:** Delete the `RecipeSearchFunction` function.
5. **Amazon CloudWatch Logs:** Delete the `/aws/lambda/RecipeSearchFunction` log group.
6. **Amazon DynamoDB:** Delete the `RecipesTable` table.
7. **AWS IAM:** Delete the `RecipesLambdaRole` role (along with its inline policies `RecipesLambdaPolicy` and `AmazonLocationGeoPlacesAccess`).

> **Note:** **Amazon Bedrock** (`apac.amazon.nova-micro-v1:0`) and **Amazon Location Service** (`geo-places` v2) are serverless pay-per-request services, so no persistent resources need to be deleted.