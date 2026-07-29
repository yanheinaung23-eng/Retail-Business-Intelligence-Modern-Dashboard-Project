# Retail Business Intelligence Modern Dashboard Project

> A modern, interactive **5-page Power BI dashboard** designed to analyze retail sales, customers, products, regions, and returns. This project demonstrates advanced Power BI development, data modeling, DAX calculations, RFM customer segmentation, and business intelligence storytelling.

---

![image](https://github.com/yanheinaung23-eng/Retail-Business-Intelligence-Modern-Dashboard-Project/blob/e17238fc83d32bff2d8809d529ced19d6db39757/Images/RETAIL%20BUSINESS%20INTELLIGENCE%20DASHBOARD%20PORTFOLIO.png)

## 📷 Video Overview

https://github.com/user-attachments/assets/a8deaa10-849d-4bb0-9a3a-c6e6dc74b3ca

## How to view in interactive mode

> Because this project is hosted on a local environment without Power BI Service web publishing, the online view is static. To experience full dashboard interactivity (filters, drill-downs, cross-highlighting, and custom tooltips), please **download the `.pbix` file** from this repository and open it in **Power BI Desktop**.


Download the pbix file [here](https://github.com/yanheinaung23-eng/Retail-Business-Intelligence-Modern-Dashboard-Project/blob/0a6e2352a78eea9449862a2d4c0164ba6309f58e/Market_Report.pbix).


## 🏗️ Data Model Architecture

![image](https://github.com/yanheinaung23-eng/Retail-Business-Intelligence-Modern-Dashboard-Project/blob/9671d12dd082d29f6c9a7018ab6914cc1ef354dd/Images/data%20model.png)

> **Architecture Overview:** The data model is designed using a **Star Schema** with a normalized **Snowflake extension** (`Regions` $\rightarrow$ `Stores`). Built in Power BI, the schema separates analytical facts from dimensional attributes to optimize query performance, simplify DAX measures, and ensure fast visual responsiveness.



Please view the datasets [here](https://github.com/yanheinaung23-eng/Retail-Business-Intelligence-Modern-Dashboard-Project/tree/24b46c03c90fb702f1ac2a3552a8aeb004228b01/Datasets).



## Advanced Analysis

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

---

## 🌎 Regional Performance & High-Value Market Analysis

![image](https://github.com/yanheinaung23-eng/Retail-Business-Intelligence-Modern-Dashboard-Project/blob/fe3543d96ae720f1077054ca798ffd4de0954428/Images/Revenue%20Trend%20by%20Sales%20Regions.png)

> **Executive Summary:** While representing only **14% of the total customer base**, the newly entered **Mexico regions** generate an exceptionally high Average Revenue Per Customer (ARPU) compared to legacy domestic markets. Combined with top-tier customer RFM scores, these markets demonstrate strong organic demand and represent the highest ROI opportunity for strategic expansion.

---

### 📐 Calculation Methodology
To evaluate customer quality and regional spending power (rather than raw transaction volume), Average Revenue Per Customer was calculated using:

$$\text{Avg. Revenue Per Customer} = \frac{\text{Total Sales Revenue}}{\text{Total Unique Customers}}$$

SQL code:
```sql
WITH region_customers AS (
SELECT 
	r.sales_region,
	ROUND(SUM(CASE WHEN t.transaction_date BETWEEN '1997-01-01' AND '1997-12-31' THEN t.quantity * p.product_retail_price ELSE 0 END)) AS sales_1997,
	ROUND(SUM(CASE WHEN t.transaction_date BETWEEN '1998-01-01' AND '1998-12-30' THEN t.quantity * p.product_retail_price ELSE 0 END)) AS sales_1998,
	COUNT(DISTINCT CASE
            WHEN t.transaction_date BETWEEN '1997-01-01' AND '1997-12-31'
            THEN t.customer_id
        END
    ) AS total_customers_1997,
	COUNT(DISTINCT CASE
			WHEN t.transaction_date BETWEEN '1998-01-01' AND '1998-12-30'
			THEN t.customer_id
		END
	) AS total_customers_1998
FROM transactions t
LEFT JOIN products p
	ON t.product_id = p.product_id
JOIN stores s
	ON t.store_id = s.store_id
JOIN regions r
	ON s.region_id = r.region_id
GROUP BY 1
)
SELECT *,
	ROUND(sales_1997 / total_customers_1997) AS avg_revenue_per_customer_1997,
	ROUND(sales_1998 / total_customers_1997) AS avg_revenue_per_customer_1998
FROM region_customers
ORDER BY sales 1
```

---

### Result

| Sales Region | 1997 Sales | 1998 Sales | 1997 Cust. | 1998 Cust. | 1997 ARPU | 1998 ARPU | Commercial Tier |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **Mexico South** | $0 | $87,241 | 0 | 98 | — | **$890** | 🔥 Highest Value per Customer |
| **Mexico Central** | $0 | $330,331 | 0 | 724 | — | **$456** | ⚡ Top Volume & Value Driver |
| **Mexico West** | $0 | $61,300 | 0 | 283 | — | **$217** | 📈 Strong Growth Potential |
| **North West** | $406,065 | $441,756 | 2,865 | 2,874 | $142 | **$154** | 🛡️ Legacy Baseline (Stable) |
| **Canada West** | $0 | $107,674 | 0 | 1,380 | — | **$78** | 👥 High Volume / Low ARPU |
| **South West** | $154,727 | $166,076 | 2,420 | 2,420 | $64 | **$69** | 🛡️ Legacy Baseline (Moderate) |
| **Central West** | $4,441 | $4,884 | 296 | 281 | $15 | **$17** | ⚠️ Low Yield |

---

### 💡 Key Findings

1. **Disproportionate Value Capture in Mexico:** Despite accounting for only **~14% of overall active customers** in 1998 (1,105 of 8,060), the three Mexico regions generated **$478,872** (~40% of total company revenue).
2. **RFM Score Validation:** Customer RFM metrics in the Mexican markets are among the highest in the entire database, confirming that the elevated ARPU is backed by real, high-intent purchasing behavior and high repeat customer value rather than one-off anomalies.
3. **ARPU Disparity:** Customers in **Mexico South ($890 ARPU)** yield **5.7x more revenue per account** than established core markets like the *North West ($154 ARPU)* and **12.8x more** than *South West ($69 ARPU)*.

---

### 🎯 Strategic Recommendations: Regional Expansion Roadmap

To maximize future revenue growth and optimize marketing/logistics expenditure, expansion capital should be allocated in the following priority order:

* **Priority 1 — Mexico South ($890 ARPU):** Maximum capitalization efficiency. Focus targeted digital campaigns and localized sales pipelines here to expand user acquisition.
* **Priority 2 — Mexico Central ($456 ARPU):** Combines strong unit economics with massive overall revenue contribution ($330K+). Scale infrastructure to capture remaining market share.
* **Priority 3 — Mexico West ($217 ARPU):** Outperforms legacy domestic averages; ideal target for localized promotions and cross-selling initiatives.

---

## 👥 Customer RFM Segmentation Analysis

![image](https://github.com/yanheinaung23-eng/Retail-Business-Intelligence-Modern-Dashboard-Project/blob/fdf7bc31987a44b959e56c5319036ca9678d2f78/Images/RFM%20scores.png)

> **Overview:** To evaluate customer behavioral loyalty and lifetime value, a custom **Recency, Frequency, and Monetary (RFM)** model was engineered in Power BI using DAX. Customers are scored on a scale from **1 to 5** across each dimension, enabling granular audience segmentation and targeted retention strategies.

---

### 📐 RFM Scoring Methodology

Each customer is evaluated across three core transactional dimensions:

* **Recency (R):** Days elapsed since the customer's last order (*Lower days = Higher engagement = Score 5*).
* **Frequency (F):** Total number of orders placed over time (*Higher order count = Stronger loyalty = Score 5*).
* **Monetary (M):** Cumulative lifetime spend (*Higher total revenue = Greater account value = Score 5*).

DAX:
```dax
// 1. Calculate Base RFM Table
Customer RFM = 
ADDCOLUMNS(
    VALUES(Customers[customer_id]),

    "Recency",
    VAR LastPurchase = 
        CALCULATE(
            MAX(Transaction_Data[transaction_date])
        )
    RETURN
        DATEDIFF(
            LastPurchase,
            [Max date],
            DAY
        ),
    
    "Frequency",
        CALCULATE(
            COUNTROWS(Transaction_Data)
        ),
    
    "Monetary",
        CALCULATE(
            SUM(Transaction_Data[Revenue])
        )
)

// 2. Score Assignment Logic
R Score = 
SWITCH(
    TRUE(),
    [Recency] <= 7, 5,
    [Recency] <= 30, 4,
    [Recency] <= 90, 3,
    [Recency] <= 180, 2,
    1
)

F Score = 
SWITCH(
    TRUE(),
    [Frequency] >= 40, 5,
    [Frequency] >= 25, 4,
    [Frequency] >= 15, 3,
    [Frequency] >= 8, 2,
    1
)

M Score = 
SWITCH(
    TRUE(),
    [Monetary] >= 500, 5,
    [Monetary] >= 250, 4,
    [Monetary] >= 120, 3,
    [Monetary] >= 60, 2,
    1
)
```

#### 🎯 Scoring Threshold Matrix

| Score | Recency (R) | Frequency (F) | Monetary (M) |
| :---: | :---: | :---: | :---: |
| **5** | $\le 7$ days | $\ge 40$ orders | $\ge \$500$ |
| **4** | $\le 30$ days | $\ge 25$ orders | $\ge \$250$ |
| **3** | $\le 90$ days | $\ge 15$ orders | $\ge \$120$ |
| **2** | $\le 180$ days | $\ge 8$ orders | $\ge \$60$ |
| **1** | $> 180$ days | $< 8$ orders | $< \$60$ |

---

### 🧩 Customer Segmentation & Mapping Logic

Using the composite RFM scores, customers are categorized into key strategic tiers for targeted marketing campaigns:

| Segment | RFM Score Pattern | Definition & Business Action |
| :--- | :---: | :--- |
| 🏆 **Champions** | `555` | **Highest-value buyers.** Highly active, frequent, and top spenders. Target with VIP rewards & early product access. |
| 🆕 **New Customers** | `511` | **Recent first-time buyers.** High recency, low transaction count/spend. Target with welcome nurture sequences. |
| 💎 **Loyal Customers** | `5` `[F ≥ 4]` `[Any M]` | **Consistent repeat buyers.** High recency and high purchase frequency. Focus on cross-selling and loyalty perks. |
| ⚠️ **At Risk** | `[R ≤ 2]` `[F ≥ 4]` `[Any M]` | **Lapsing high-frequency buyers.** Low recency despite high historical order count. Trigger win-back & discount offers. |
| ❌ **Lost Customers** | `111` | **Inactive low-value buyers.** Lowest engagement across all metrics. Re-engage via low-cost automated email flows. |
| ⚙️ **Others** | *All Other Profiles* | **General Customer Base.** Standard promotional and seasonal campaigns. |

---

## 📦 Product ABC Classification & Inventory Analysis

![image](https://github.com/yanheinaung23-eng/Retail-Business-Intelligence-Modern-Dashboard-Project/blob/b0f0a5668cfe6358c405d8c75d701f61313b6b5c/Images/ABC%20Product%20Classification.png)

> **Executive Summary:** Using the **Pareto Principle (80/20 Rule)**, products were categorized into A, B, and C tiers based on cumulative revenue impact. The analysis revealed high revenue fragmentation, with **62.6% of the product catalog categorized under Class A**. This lack of clear "Hero" SKUs ties up working capital and increases operational complexity across inventory management.

---

### 📐 Classification Methodology & Thresholds

Products are ranked from highest to lowest revenue contribution and segmented using cumulative revenue share cutoffs:

| Class | Revenue Share % | Cumulative Cutoff | Operational Meaning | Strategic Focus |
| :---: | :---: | :---: | :--- | :--- |
| **Class A** | Top 80% | $\le 80\%$ | High-Impact Drivers | Priority inventory & max availability |
| **Class B** | Next 15% | $80\% - 95\%$ | Mid-Tier Performers | Standard stock control & monitoring |
| **Class C** | Next 5% | $> 95\%$ | Long-Tail Items | Minimal holding & potential rationalization |

DAX:
```dax
// Product ABC Classification
ABC Class = 
SWITCH(
    TRUE(),
    [Cumulative Revenue %] <= 0.80, "A",
    [Cumulative Revenue %] <= 0.95, "B",
    "C"
)
```

---

### 💡 Key Findings

1. **No Clear "Hero" Products (Catalog Fragmentation):** Class A revenue is distributed across **62.6%** of the product catalog. Rather than having a concentrated set of top-performing flagship SKUs, revenue is spread thinly across a broad range of brands (e.g., *Hermanos*, *Tell Tale*, *Ebony*, *Tri-State*).
2. **Operational & Cash Flow Strain:** Treating **62.2%–62.6%** of total inventory with Class A priority creates significant supply chain overhead, inflates carrying costs, and locks up critical working capital in holding stock.
3. **Margin Masking Risk:** Classifying SKUs solely by gross revenue hides profitability differences; low-margin items that generate high volume are currently receiving the same replenishment priority as high-margin items.

---

### 🎯 Strategic Recommendations

* **1. Margin-Based Optimization:** Cross-analyze Class A items against net profit margins to identify true high-margin revenue drivers and elevate them into flagship "Hero" product status.
* **2. Marketing & Visual Placement:** Direct marketing expenditure and prime digital storefront placement toward high-margin Class A products to accelerate their volume velocity.
* **3. Inventory Policy Realignment:** Reclassify lower-margin Class A items into tighter reorder schedules to reduce holding costs and optimize cash flow efficiency.

---

## 📌 Conclusion & Executive Summary

> **Macro Takeaway:** This end-to-end Business Intelligence project demonstrates how combining **SQL metric decomposition**, **DAX-driven RFM modeling**, and **Pareto inventory classification** transforms raw transactional data into high-value commercial strategy. The analysis proves that while top-line revenue doubled (**+112.2% YoY**), future sustainable profitability depends on prioritizing high-ARPU international markets and rationalizing product inventory.

---

### 🔑 Summary of Core Insights

1. **Acquisition & International Expansion Drove Revenue Doubling:**  
   The primary catalyst behind the **+112.2% revenue surge** (growing from **$565K to $1.2M**) was a **+44.4% expansion in active customers**, directly unlocked by entering 4 new regions in Year 2.

2. **Mexico Represents an Exceptional Growth Opportunity:**  
   The regional analysis highlights a significant yield disparity: despite representing only **14% of the customer base**, the Mexico markets generated **~$479K (~40% of total company revenue)**. Customers in **Mexico South ($890 ARPU)** yield **5.7x to 12.8x more revenue** than legacy domestic accounts.

3. **Inventory Management Requires Profitability Alignment:**  
   While top-line growth is strong, product portfolio analysis reveals that **62.6% of the catalog falls into Class A**. Treating over six out of ten SKUs as high-priority drivers creates supply chain strain and ties up working capital in low-margin inventory.

---

### 🎯 Consolidated Strategic Action Plan

| Pillar | Strategic Goal | Operational Action | Expected Business Outcome |
| :--- | :--- | :--- | :--- |
| **1. Geographic Growth** | Capitalize on High-ARPU Markets | Reallocate marketing capital to **Mexico South** and **Mexico Central**. | Higher ROI per customer acquisition dollar and elevated total account value. |
| **2. Customer Retention** | Protect & Nurture High Tiers | Deploy automated email/VIP nurture sequences for **Champions (`555`)** and win-back flows for **At Risk (`R≤2, F≥4`)** accounts. | Higher customer lifetime value (LTV) and increased purchase frequency. |
| **3. Catalog Rationalization** | Eliminate Inventory Bottlenecks | Cross-reference Class A products against net profit margins; prioritize high-margin "Hero" SKUs for prime digital placement. | Optimized cash flow, reduced holding costs, and improved profit margins. |

---

### 🛠️ Technical Capabilities Demonstrated

* **Data Architecture:** Designed a scalable **Star Schema** with a **Snowflake extension** (`Regions` $\rightarrow$ `Stores`) in Power BI.
* **Advanced Analytics:** Applied **SQL driver decomposition** ($\text{Revenue} = \text{Customers} \times \text{Frequency} \times \text{AOV}$) to quantify growth catalysts.
* **Custom DAX Modeling:** Engineered dynamic **RFM scoring** algorithms and cumulative revenue **Pareto classification (80/20 Rule)**.
* **Executive Visualization:** Built an interactive **5-page Power BI reporting suite** tailored for executive decision-making.

---
### 👨‍💻 Author

Yan Hein Aung

⭐ If you found this project helpful, feel free to star the repository!
