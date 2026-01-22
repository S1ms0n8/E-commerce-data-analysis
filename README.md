# 📊 Olist E-commerce Strategic Research: A Multi-Dimensional SQL Study

## 📖 Research Story
This project is an **independent research study** based on the Olist Brazilian E-commerce dataset. I treated this as a strategic mission to solve a core e-commerce dilemma: **How do operational efficiency and customer segmentation directly impact business growth and brand perception?**

By building a framework to connect logistics, finance, and B2B sales performance, I moved beyond simple data extraction to find actionable business intelligence using advanced SQL.

## 📂 Data Source
The dataset used for this research is the **Brazilian E-Commerce Public Dataset by Olist**, which can be found on Kaggle:
👉 [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

This dataset contains information on 100k orders made between 2016 and 2018 across multiple marketplaces in Brazil. Its real-world nature allows for a deep dive into genuine customer behavior and operational challenges.

License:👉https://creativecommons.org/licenses/by-nc-sa/4.0/



---

## 🎯 Research Objectives & Business Impact

| What we study? | Why it matters? (Consequences) |
| :--- | :--- |
| **Customer Segmentation (RFM)** | **Consequence:** Identifying "Champions" for high-ROI loyalty and "At Risk" customers for win-back campaigns. |
| **Logistics vs. Satisfaction** | **Consequence:** Proving how delivery speed affects ratings to manage expectations and courier contracts. |
| **B2B Sales Funnel** | **Consequence:** Understanding conversion per segment to prioritize the sales team's efforts on high-value leads. |
| **Payment & Financing** | **Consequence:** Analyzing installment dependency to optimize payment options and increase Average Order Value (AOV). |

---

## 💻 Technical Analysis (Full SQL Codebase)

### 1. Advanced RFM Segmentation & Customer Retention
*Categorizing customers by Recency, Frequency and Monetary value.*

```sql


With customer_metrics AS(
    SELECT
        c.customer_unique_id,
        Max(o.order_purchase_timestamp) as Recency,
        count(o.order_id) as Frequency,
        SUM(p.payment_value) as Monetary
        from customers as c 
        join orders as o on c.customer_id=o.customer_id
        join order_payments as p on p.order_id=o.order_id
        where o.order_status= 'delivered'
        group by 1
    
),
rfm_scores AS(
    SELECT
        customer_unique_id,
        Recency,
        Frequency,
        Monetary,
        ntile(5) over (order by Recency asc) as r_score,
        ntile(5) over (order by Frequency asc) as f_score,
        ntile(5) over (order by Monetary asc) as m_score
    from customer_metrics
),
segments AS(
    SELECT *,
    CASE 
        WHEN r_score >= 4 AND (f_score + m_score) >= 8 THEN  'Champions'
        WHEN r_score >= 4 AND (f_score + m_score) BETWEEN 5 AND 7 THEN 'Potencial Loyalists'
        WHEN r_score >= 4 AND (f_score + m_score) < 5 THEN 'New Customers'
        WHEN r_score BETWEEN 2 AND 3 AND (f_score + m_score) >=6 THEN 'Loyal Customers'
        WHEN r_score BETWEEN 2 AND 3 AND (f_score + m_score) BETWEEN 3 AND 5 THEN 'Needs Attention'
        WHEN r_score < 2 AND (f_score + m_score) >= 6 THEN 'Cant Lose Them'
        WHEN r_score < 2 AND (f_score + m_score) BETWEEN 3 AND 5 THEN 'Hibernating'
        ELSE 'Lost' 
    END AS rfm_segment
    from rfm_scores
)
SELECT 
    rfm_segment,
    COUNT(*) as customer_count,
    round(avg(Monetary),2) as avg_monetery,
    round(avg(julianday((SELECT Max(Recency) from rfm_scores)) - julianday(Recency)),0) as avg_recency_days
    FROM segments
    GROUP BY 1
    ORDER BY avg_monetery DESC;
    
```

### 2.Retntion Analysis (Repeat VS One-Time)
*Identifying the share of loyal customers returning to the platform.*

```sql
With orders_per_customer AS(
    select c.customer_unique_id, count(o.order_id) as order_count
    FROM customers as c
    JOIN orders as o on o.customer_id=c.customer_id
    where order_status= 'delivered'
    GROUP BY 1
)
Select 
    CASE 
        WHEN order_count > 1  THEN 'Repeat Customers' 
        ELSE 'One-Time Customers' 
    END as customer_type,
    count(*) as count,
    round(100.0 * count(*) / (SELECT count(*) from orders_per_customer),2) as percentage
    FROM orders_per_customer
    GROUP BY 1;
```
###  3. Logistics & Satisfaction (Research Hypothesis)
*Measuring how delivery speed and delays impact scores.*
```sql
SELECT 
    ore.review_score,
    round(avg(julianday(o.order_delivered_customer_date) - julianday(o.order_purchase_timestamp)),1) as actual_delivery_days,
    round(avg(julianday(o.order_delivered_customer_date) - julianday(o.order_estimated_delivery_date)),1) as delay_vs_estimate
from orders as o
join order_reviews as ore on ore.order_id= o.order_id
where o.order_status= 'delivered' AND o.order_delivered_customer_date is NOT NULL
GROUP BY 1
ORDER BY 1 DESC;
```
###  4. B2B Sales Funnel Efficiency
*Analyzing which business segments have the best conversion and revenue potential*
```sql
SELECT
    lq.origin,
    lc.business_segment,
    count(lq.mql_id) as total_leads,
    count(lc.mql_id) as closed_deals,
    round(cast(count(lc.mql_id) as float) / count(lq.mql_id) * 100,2) as conv_rate
FROM leads_qualified as lq
LEFT JOIN leads_closed as lc on lc.mql_id=lq.mql_id
GROUP BY 1,2
HAVING closed_deals > 0
ORDER BY conv_rate DESC;
```
###  5. Payment Preferences
*Breaking down revenue by payment type and analyzing installment behavior.*
```sql
SELECT 
    payment_type,
    ROUND(AVG(payment_installments), 1) AS avg_installments,
    SUM(payment_value) AS total_revenue,
    ROUND(100.0 * SUM(payment_value) / (SELECT SUM(payment_value) FROM order_payments), 2) AS rev_share_pct
FROM order_payments
GROUP BY 1 ORDER BY total_revenue DESC;
```
## 📈 Strategic Insights (Based on Real Data Results)

### 1. RFM & Retention: The Loyalty Gap

* **The Retention Challenge**: A massive **97.0% (90,557)** of customers purchased only once. Only **3.0% (2,801)** are repeat customers.

* **The Core Asset**: **14,781 Champions** drive the marketplace with an average spend of **313.16**.

* **The High-Value Leak**: **10,933 customers** in the "Cant Lose Them" segment have a high historical spend **(238.33)** but haven't purchased in **473 days**.

### 2. Logistics: The "Retention Killer"

* **The 5-Star Benchmark**: Top-rated orders arrived in **10.7 days**, usually **12.7 days** before the estimated deadline.

* **The 1-Star Experience**: Dissatisfied customers waited an average of **21.3 days**—twice as long as 5-star customers.

* **Broken Promises**: 1-star reviews correlate with a narrow "safety buffer" of only **3.4 days** before the estimate.

### 3. B2B Sales Funnel Efficiency

* **Winning Segments**: **Home Decor** leads via `organic_search` (44 leads) and **Car Accessories** via `paid_search` (20 leads) show a **100%** **conversion rate**.

* **High-Volume Drivers**: `organic_search` is the strongest source for **Audio/Electronics** and **Health/Beauty**, consistently converting at **100%**.

### 4. Payment Preferences & Financing

* **Dominant Method**: **Credit cards** account for **78.34%** of total revenue **(12.5M)**, with an average of **3.5** installments.

* **Cash Alternative**: **Boleto** represents **17.92%** of revenue, serving as the primary non-credit option.

## 🚀 Final Conclusions & Recommendations
* **Retention Focus**: Implement **win-back campaign**s for the **10,933 "Cant Lose Them"** customers. Reactivating this high-spend group is more cost-effective than acquiring new users.

* **The 11-Day Logistics Goal**: To maximize 5-star ratings, Olist should aim for a **"Golden Window" of 10.7 days**. Deliveries exceeding 14 days significantly damage NPS and repeat purchase probability.

* **B2B Strategy**: Double down on **Organic and Paid Search** for **Health/Beauty and Home Decor**, as these show the highest efficiency in closing deals.

## 🛠️ Tech Stack
* **SQL Engine**: **SQLite** / **PostgreSQL**

* **Key Functions**: Window Functions (`NTILE`), CTEs, Date Math (`julianday`), Data Aggregation.

Created as a showcase of technical SQL proficiency and data-driven business strategy.
