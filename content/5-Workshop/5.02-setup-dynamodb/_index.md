---
title: "Create DynamoDB Database"
date: 2026-11-27
weight: 2
chapter: false
pre: "<b>5.2. </b>"
---

#### Implementation Steps

1. Navigate to the **AWS Management Console**, search for **DynamoDB**, and select **Create table**.
2. Configure the table settings:
  - **Table name:** `RecipesTable`
  - **Partition key:** `RecipeID` (Data type: `String`)
  - **Table settings:** Select **Default settings** (On-demand mode).
3. Click **Create table** and wait for the table status to become **Active**.
4. Open `RecipesTable` → **Explore table items** → **Create item** to populate sample data with the following attributes: `RecipeID`, `Name`, `Category`, `Ingredients`, `EstimatedCostVND`, `CookingTimeMinutes`, `MoodTags`, `DietTags`, `CookingSteps`.

![anh2](/images/5-Workshop/5.02-setup-dynamodb/anh2.png)

![anh3](/images/5-Workshop/5.02-setup-dynamodb/anh3.png)
