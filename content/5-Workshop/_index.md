---
title: "Workshop"
date: 2026-11-27
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Building Smart Kitchen: Serverless Recipe Suggestion & AI Chef Workshop on AWS

#### Overview

In this workshop, you will build and deploy **Smart Kitchen (Bếp Thông Minh)** — a complete serverless, event-driven web application on AWS that provides personalized recipe recommendations, AI-powered culinary counseling via Generative AI, and geospatial grocery store lookup with budget guidance.

The architecture completely decouples the static frontend from the serverless backend to achieve high scalability, pay-as-you-go cost optimization, and security over HTTPS:

+ **Amazon S3 & Amazon CloudFront** — Static frontend hosting (`template.html`) with global HTTPS delivery enabling HTML5 Geolocation API (`navigator.geolocation`)
+ **Amazon API Gateway (HTTP API)** — Exposes RESTful endpoints (`GET /search`, `GET /nearby-stores`, `POST /chat`) with CORS enabled
+ **AWS Lambda (Python 3.12)** — Serverless backend (`RecipeSearchFunction`) handling ingredient matching, nutrition/budget filtering, AI prompts, and geolocation lookups
+ **Amazon DynamoDB** — Stores recipe catalog data (`RecipesTable`) in On-Demand mode
+ **Amazon Bedrock** — Powers the empathetic AI Chef Chatbot using **Amazon Nova Micro** (`apac.amazon.nova-micro-v1:0`)
+ **Amazon Location Service (`geo-places` v2)** — Geocodes manually entered addresses and searches nearby grocery stores ranked by distance and budget tier
+ **AWS IAM & Amazon CloudWatch** — Least-privilege execution role (`RecipesLambdaRole`) and centralized application logging

#### Contents

1. [Overview & Preparation](5.01-workshop-overview/)
2. [Create DynamoDB Database](5.02-setup-dynamodb/)
3. [Configure AWS IAM Role](5.03-configure-iam-role/)
4. [Deploy AWS Lambda Backend](5.04-deploy-lambda-backend/)
5. [Set Up Amazon API Gateway](5.05-configure-api-gateway/)
6. [Deploy Frontend to S3 & CloudFront](5.06-deploy-s3-cloudfront/)
7. [Application Testing](5.07-test-application/)
8. [Resource Cleanup](5.08-cleanup/)
