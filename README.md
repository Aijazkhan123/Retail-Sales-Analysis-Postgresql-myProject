# Retail Sales Analysis — SQL Project

## 1. Project Overview
This project documents an end-to-end  Data Analysis and data-cleaning workflow performed on a retail point-of-sale dataset using SQL. Built within the `Retail` database using the `retail_sales` fact table, the workflow encompasses relational table creation, data hygiene, ten business intelligence queries, and operational conclusions.

### Project Objectives
* Design and create the `retail_sales` fact table to store order-line transactions, customer demographics, and financial figures.
* Clean and standardize raw point-of-sale records by removing incomplete entries and standardizing categorical labels.
* Answer ten targeted business intelligence questions covering daily sales, product category performance, customer demographics, seasonal trends, and store shift activity.
* Translate analytical findings into actionable operational and marketing decisions.

### Tools & SQL Concepts Used
* **Database & Language:** MySQL / SQL (`Retail` database)
* **Data Definition & Manipulation (DDL/DML):** `DROP TABLE`, `CREATE TABLE`, `DELETE`, `UPDATE`
* **Filtering & Sorting:** `WHERE`, comparison and logical operators (`AND`, `OR`, `IS NULL`), `ORDER BY`, `LIMIT`
* **Aggregations & Grouping:** `SUM()`, `COUNT()`, `COUNT(DISTINCT)`, `AVG()`, `ROUND()`, `GROUP BY`
* **Date & Time Functions:** `EXTRACT(YEAR FROM ...)`, `EXTRACT(MONTH FROM ...)`, date and time range filtering
* **Conditional Logic:** `CASE WHEN ... THEN ... ELSE ... END`
* **Advanced Querying:** Common Table Expressions (`WITH`), Window Functions (`RANK() OVER (PARTITION BY ... ORDER BY ...)`)

---

## 2. Dataset Description & Schema
The analytical model relies on the `retail_sales` fact table inside the `Retail` database, capturing order-line transactions, customer demographics, and financial figures across three product categories (`Clothing`, `Beauty`, and `Electronics`).

| Column Name | Data Type | Constraint | Description |
| :--- | :--- | :--- | :--- |
| `transaction_id` | `INT` | `PRIMARY KEY` | Unique identifier for each transaction |
| `sale_date` | `DATE` | — | Date the transaction took place |
| `sale_time` | `TIME` | — | Time the transaction took place (24-hour format) |
| `customer_id` | `INT` | — | Unique identifier assigned to each customer |
| `gender` | `VARCHAR(15)` | — | Standardized customer gender (`Male`, `Female`) |
| `age` | `INT` | — | Customer age in years |
| `category` | `VARCHAR(15)` | — | Product category (`Clothing`, `Beauty`, `Electronics`) |
| `price_per_unit` | `FLOAT` | — | Retail selling price per single unit |
| `quantity` | `INT` | — | Number of units purchased in the order |
| `cogs` | `FLOAT` | — | Cost of Goods Sold |
| `total_sale` | `FLOAT` | — | Total revenue generated from the transaction |

### Table Creation Script
```sql
DROP TABLE IF EXISTS retail_sales;

CREATE TABLE retail_sales
(
    transaction_id INT PRIMARY KEY,
    sale_date DATE,
    sale_time TIME,
    customer_id INT,
    gender VARCHAR(15),
    age INT,
    category VARCHAR(15),
    price_per_unit FLOAT,
    quantity INT,
    cogs FLOAT,
    total_sale FLOAT
);
```

---

## 3. Data Cleaning & Preparation
Prior to analysis, the dataset was audited and standardized through two primary hygiene steps:
1. **Null Value Removal:** Audited all columns for missing entries and deleted incomplete rows so revenue totals and demographic averages remain accurate.
2. **Gender Standardization:** Replaced abbreviated codes (`'M'` and `'F'`) with full descriptive labels (`'Male'` and `'Female'`).

```sql
-- 1. Remove records containing NULL values across any column
DELETE FROM retail_sales
WHERE transaction_id IS NULL
   OR sale_date IS NULL
   OR sale_time IS NULL
   OR customer_id IS NULL
   OR gender IS NULL
   OR age IS NULL
   OR category IS NULL
   OR quantity IS NULL
   OR price_per_unit IS NULL
   OR cogs IS NULL
   OR total_sale IS NULL;

-- 2. Standardize gender values
UPDATE retail_sales SET gender = 'Male' WHERE gender = 'M';
UPDATE retail_sales SET gender = 'Female' WHERE gender = 'F';
```

---

## 4. Business Questions, SQL Queries & Business Value

### Q1. Sales on November 5, 2022
**Business Question:** Retrieve all columns for sales made on November 5, 2022.
```sql
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';
```
**Business Value:** Isolating a single trading day enables operations teams to inspect daily order volume, basket sizes, and category mix to evaluate daily promotions or detect stockouts.

---

### Q2. High-Quantity Clothing Transactions in November 2022
**Business Question:** Retrieve all transactions where the category is `Clothing`, quantity sold is greater than 2, and the transaction occurred in November 2022.
```sql
SELECT *
FROM retail_sales
WHERE category = 'Clothing'
  AND quantity > 2
  AND sale_date >= '2022-11-01'
  AND sale_date < '2022-12-01'
ORDER BY quantity;
```
**Business Value:** Multi-unit apparel purchases during November indicate seasonal and early holiday demand. Bundling complementary clothing items or offering multi-buy incentives during peak months can lift average order value.

---

### Q3. Total Sales Revenue & Order Count by Category
**Business Question:** Calculate total sales revenue and the total number of transactions for each product category.
```sql
SELECT
    category,
    SUM(total_sale) AS net_sales,
    COUNT(transaction_id) AS total_orders
FROM retail_sales
GROUP BY category;
```
**Business Value:** Comparing `net_sales` alongside `total_orders` clarifies whether a category succeeds via high transaction frequency or higher unit pricing, guiding shelf-space allocation and promotional budgets.

---

### Q4. Average Customer Age in the Beauty Category
**Business Question:** Find the average age of customers who purchased products from the `Beauty` category.
```sql
SELECT
    ROUND(AVG(age), 2) AS avg_customer_age
FROM retail_sales
WHERE category = 'Beauty';
```
**Business Value:** Establishing the average buyer age in the Beauty category provides a clear demographic benchmark for product curation, merchandising, and targeted marketing campaigns.

---

### Q5. High-Value Transactions Above 1,000
**Business Question:** Retrieve all transactions where total sales exceeded 1,000.
```sql
SELECT *
FROM retail_sales
WHERE total_sale > 1000
ORDER BY total_sale;
```
**Business Value:** High-ticket orders exceeding 1,000 are driven by premium unit prices combined with multi-unit quantities. Monitoring these transactions helps identify high-margin product lines and big-ticket buying patterns.

---

### Q6. Transactions by Gender and Category
**Business Question:** Calculate the total number of transactions made by each gender in each product category.
```sql
SELECT
    gender,
    category,
    COUNT(transaction_id) AS total_transactions
FROM retail_sales
GROUP BY gender, category;
```
**Business Value:** Cross-tabulating gender and product category highlights demand distribution across customer segments, enabling tailored promotions and cross-selling strategies for each group.

---

### Q7. Best-Selling Month in Each Year
**Business Question:** Calculate total monthly sales for each year and identify the best-selling month in each year.
```sql
WITH monthly_sales AS (
    SELECT
        EXTRACT(YEAR FROM sale_date) AS year,
        EXTRACT(MONTH FROM sale_date) AS month,
        SUM(total_sale) AS total_sales,
        RANK() OVER (
            PARTITION BY EXTRACT(YEAR FROM sale_date)
            ORDER BY SUM(total_sale) DESC
        ) AS rnk
    FROM retail_sales
    GROUP BY 1, 2
)
SELECT *
FROM monthly_sales
WHERE rnk = 1;
```
**Business Value:** Ranking monthly revenue within each calendar year pinpoints peak seasonal months so inventory procurement, marketing budgets, and store staffing can be scaled up ahead of recurring demand surges.

---

### Q8. Top 5 Customers by Total Sales
**Business Question:** Identify the top 5 customers based on their highest cumulative sales.
```sql
SELECT
    customer_id,
    SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sales DESC
LIMIT 5;
```
**Business Value:** The top five customers contribute disproportionate revenue and represent ideal candidates for VIP loyalty programs, early access to new collections, and dedicated retention outreach.

---

### Q9. Unique Customers by Category
**Business Question:** Calculate the number of unique customers who purchased products from each category.
```sql
SELECT
    category,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM retail_sales
GROUP BY category
ORDER BY unique_customers DESC;
```
**Business Value:** Counting distinct `customer_id` entries measures true customer reach rather than repeat order volume, revealing how broadly each category attracts buyers and where cross-category upselling can grow.

---

### Q10. Order Count by Store Shift
**Business Question:** Categorize transactions into Morning (`< 12:00`), Afternoon (`12:00–17:59`), and Evening (`≥ 18:00`) shifts and count the orders in each shift.
```sql
SELECT
    CASE
        WHEN sale_time < '12:00:00' THEN 'MORNING'
        WHEN sale_time < '18:00:00' THEN 'AFTERNOON'
        ELSE 'EVENING'
    END AS shift,
    COUNT(*) AS total_orders
FROM retail_sales
GROUP BY shift
ORDER BY total_orders DESC;
```

Conclusion
Cleaning the retail_sales dataset and analyzing it across time, product categories, customer demographics, and operational shifts converts raw point-of-sale records into actionable business decisions across merchandising, staffing, inventory planning, and customer retention.

---

## Author
Aijaz Ahmed



