# Retail Business Intelligence Modern Dashboard Project

> A modern, interactive **5-page Power BI dashboard** designed to analyze retail sales, customers, products, regions, and returns. This project demonstrates advanced Power BI development, data modeling, DAX calculations, RFM customer segmentation, and business intelligence storytelling.

---

![image](https://github.com/yanheinaung23-eng/Retail-Business-Intelligence-Modern-Dashboard-Project/blob/e17238fc83d32bff2d8809d529ced19d6db39757/Images/RETAIL%20BUSINESS%20INTELLIGENCE%20DASHBOARD%20PORTFOLIO.png)

## 📷 Video Overview

https://github.com/user-attachments/assets/a8deaa10-849d-4bb0-9a3a-c6e6dc74b3ca

## Advanced Analysis using SQL

## 📈 1. Sales Jump & Root Cause Analysis

![image](https://github.com/yanheinaung23-eng/Retail-Business-Intelligence-Modern-Dashboard-Project/blob/2acda0759caa4d4bb7924d2d9ca8ba22419a0166/Images/Sales%20jump%20root%20cause.png)

> **Executive Summary:** Top-line revenue surged by **+112.2% YoY**, elevating average monthly revenue from **~$50K (Year 1)** to **~$100K (Year 2)**. A SQL-driven metric decomposition identified **Customer Acquisition (+44.4%)** as the primary driver, directly fueled by geographic expansion into **4 new international regions**.

---

### 🔍 Analytical Methodology
To isolate the exact levers behind the revenue surge, a SQL driver decomposition framework was applied using the core commercial valuation equation:

$$\text{Revenue} = \text{Unique Customers} \times \text{Purchase Frequency} \times \text{Average Order Value (AOV)}$$

SQL Code:
```sql
WITH transaction_summary AS (
SELECT 
	CASE 
		WHEN t.transaction_date BETWEEN '1997-01-01' AND '1997-12-31' THEN '1997'
		WHEN t.transaction_date BETWEEN '1998-01-01' AND '1998-12-30' THEN '1998'
	END AS time_period,
	COUNT(DISTINCT CONCAT(t.customer_id, '-', t.store_id, '-', t.transaction_date)) AS total_transactions,
    COUNT(DISTINCT t.customer_id) AS unique_customers,
    SUM(t.quantity * p.product_retail_price) AS total_revenue
FROM transactions t
LEFT JOIN products p ON t.product_id = p.product_id
GROUP BY time_period
)
SELECT
    time_period,
    ROUND(total_revenue) AS total_revenue,
    total_transactions,
    unique_customers,
 	ROUND(total_revenue / NULLIF(total_transactions, 0)) AS AOV,
    ROUND(total_transactions / NULLIF(unique_customers, 0)) AS avg_frequency
FROM transaction_summary
ORDER BY time_period ASC;
```
### Result

| Time Period | Total Revenue | Total Transactions | Unique Customers | AOV | Avg. Frequency |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **1997** | $565,233 | 20,530 | 5,581 | $28 | 3 |
| **1998** | $1,199,262 | 37,851 | 8,060 | $32 | 4 |

---

### 💡 Key Findings

**1. Strong Customers Acquisition (+44.4%)**

- The customer base expanded significantly from 5,581 to 8,060 active customers. This shows strong marketing momentum and market expansion.

**2. Improved Customer Retention & Habit (+33.3%)**

- Average purchase frequency increased from ~3 to ~4 orders per customer per year. Existing customers are coming back more often, indicating stronger product-market fit.

**3. Average Order Value Growth (+14.3%)**

- While still positive, AOV showed the slowest growth among all drivers (+14.3%). Customers are buying more often.

---

### 🎯 Strategic Takeaways & Validation

1. **Acquisition-Driven Expansion:** The 44.4% increase in unique customers served as the primary growth engine, proving that top-line performance was driven by volume expansion.
2. **Geographic Validation:** Regional trend analysis confirms that entering **4 new regions with 2 new international markets in Year 2** directly unlocked this new customer segment, validating the commercial success of the international go-to-market strategy.
