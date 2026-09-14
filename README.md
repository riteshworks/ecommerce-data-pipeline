# Shopify to PostgreSQL: Automated ETL & Customer Segmentation

## 📖 Overview
Managing historical Shopify order data in Excel is painful, especially when dealing with thousands of rows, nested line items, and complex refunds. This project demonstrates an automated ETL (Extract, Transform, Load) pipeline built with **n8n** to pull historical order data from the Shopify GraphQL API, parse complex JSONL, and upsert it into a **PostgreSQL** database.

Once the data is in PostgreSQL, it is queried to generate actionable customer segmentation for targeted marketing campaigns (e.g., Winback, Feedback, Reactivation) using advanced SQL.

## 🛠️ Tech Stack
*   **Automation:** n8n
*   **Data Source:** Shopify GraphQL Admin API (Bulk Operations)
*   **Database:** PostgreSQL
*   **BI / Visualization:** Metabase (or any SQL-compatible BI tool)
*   **Language:** JavaScript (inside n8n), SQL

## 🏗️ Pipeline Architecture (n8n)
The workflow (`workflow.json`) automates the entire data sync process:

1.  **Trigger:** Manual start or scheduled.
2.  **Trigger Bulk Operation:** Initiates a Shopify GraphQL Bulk Operation to extract all historical orders.
3.  **Polling:** Waits 60 seconds, checks the bulk operation status, and loops until `COMPLETED`.
4.  **Download:** Downloads the resulting JSONL file containing Orders, LineItems, Transactions, and DiscountApplications.
5.  **Parse & Clean:** A custom JavaScript node parses the JSONL file, maps relationships between Orders and LineItems, calculates refunds, and extracts variant options (Size/Color) from SKUs or titles.
6.  **Upsert:** Batches the cleaned data (1500 items per batch) and performs a PostgreSQL `UPSERT` to keep the database in sync without duplicating records.

## 📊 Customer Segmentation Logic (SQL)
The `customer_segmentation.sql` file contains a complex query that aggregates customer data to calculate:
*   LTV (Lifetime Value) and Net LTV
*   Total Orders and Quantity
*   Cancellation Rates
*   Average Days Between Orders
*   Days Since Last Order

Based on these metrics, the query outputs segments that can be used for email marketing campaigns:

| Segment | Logic |
| :--- | :--- |
| **Feedback Only** | Cancellation rate > 30% |
| **Feedback + Incentive** | Cancellation rate between 15% and 30% |
| **Reactivate + Feedback** | Cancellation rate < 15% |
| **Winback Campaign** | 0% Cancellation + Last ordered 100–150 days ago |
| **Reminder** | 0% Cancellation + Last ordered 50–93 days ago |
| **Strong Coupon** | 0% Cancellation + Last ordered 150+ days ago |

## 🚀 Setup Instructions

### Prerequisites
*   A running instance of n8n.
*   A PostgreSQL database.
*   A Shopify Custom App with `read_orders` access.

### 1. Database Setup
Run the SQL snippet in the files to create the RFM segmentation.
