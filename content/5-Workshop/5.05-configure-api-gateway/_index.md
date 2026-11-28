---
title: "Set Up Amazon API Gateway"
date: 2026-11-27
weight: 5
chapter: false
pre: "<b>5.5. </b>"
---

#### Implementation Steps

1. Navigate to **API Gateway → Create API → HTTP API → Build**.
2. Click **Add integration** → select **Lambda** → point it to `RecipeSearchFunction` and name the API `BepThongMinhAPI`.

![lambda](/images/5-Workshop/5.05-configure-api-gateway/lambda.png)

3. Configure 3 **Routes**, all pointing to `RecipeSearchFunction`:
  - `GET /search`
  - `GET /nearby-stores`
  - `POST /chat`

![routes](/images/5-Workshop/5.05-configure-api-gateway/routes.png)

4. Name the Stage `prod` and complete the creation.

![prod](/images/5-Workshop/5.05-configure-api-gateway/prod.png)

5. Navigate to the **CORS** section of the API and configure:
  - **Access-Control-Allow-Origin:** `*`
  - **Access-Control-Allow-Methods:** `GET, POST, OPTIONS`
  - **Access-Control-Allow-Headers:** `Content-Type, Authorization`

![cors](/images/5-Workshop/5.05-configure-api-gateway/cors.png)

6. Save the **Invoke URL** (`https://1f10tq33a6.execute-api.ap-southeast-1.amazonaws.com/prod`) to assign to the `API_BASE` variable in the Frontend.
