# 📊 Olist E-commerce Strategic Research: A Multi-Dimensional SQL Study

## 📖 Research Story
This project is an **independent research study** based on the Olist Brazilian E-commerce dataset. I treated this as a strategic mission to solve a core e-commerce dilemma: **How do operational efficiency and customer segmentation directly impact business growth and brand perception?**

By building a framework to connect logistics, finance, and B2B sales performance, I moved beyond simple data extraction to find actionable business intelligence using advanced SQL.

## 📂 Data Source
The dataset used for this research is the **Brazilian E-Commerce Public Dataset by Olist**, which can be found on Kaggle:
👉 [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

This dataset contains information on 100k orders made between 2016 and 2018 across multiple marketplaces in Brazil. Its real-world nature allows for a deep dive into genuine customer behavior and operational challenges.



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
*Objective: Categorize the 100k+ customer base and identify loyalty patterns.*

```sql
-- 1A: RFM Segmentation Logic
WITH base_metrics AS (
    SELECT 
        c.customer_unique_id,
        MAX(o.order_purchase_timestamp) AS last_purchase,
        COUNT(o.order_id) AS total_orders,
        SUM(p.payment_value) AS total_spent
    FROM customers c
    JOIN orders o USING (customer_id)
    JOIN order_payments p USING (order_id)
    WHERE o.order_status = 'delivered'
    GROUP BY 1
),
rfm_scores AS (
    SELECT 
        customer_unique_id,
        last_purchase,
        NTILE(5) OVER (ORDER BY last_purchase ASC) AS r_score, 
        NTILE(5) OVER (ORDER BY total_orders ASC) AS f_score,     
        NTILE(5) OVER (ORDER BY total_spent ASC) AS m_score      
    FROM base_metrics
),
segments AS (
    SELECT *,
        CASE 
            WHEN r_score >= 4 AND (f_score + m_score) >= 8 THEN 'Champions'
            WHEN r_score >= 4 AND (f_score + m_score) BETWEEN 5 AND 7 THEN 'Potential Loyalists'
            WHEN r_score < 2 AND (f_score + m_score) >= 6 THEN 'Cant Lose Them'
            WHEN r_score < 2 AND (f_score + m_score) BETWEEN 3 AND 5 THEN 'Hibernating'
            ELSE 'Regular'
        END AS rfm_segment
    FROM rfm_scores
)
SELECT rfm_segment, COUNT(*) AS count, ROUND(AVG(total_spent), 2) AS avg_spent
FROM segments GROUP BY 1 ORDER BY avg_spent DESC;

-- 1B: Retention Rate (Repeat vs One-Time)
WITH orders_per_customer AS (
    SELECT customer_unique_id, COUNT(order_id) AS order_count
    FROM customers
    JOIN orders USING (customer_id)
    WHERE order_status = 'delivered'
    GROUP BY 1
)
SELECT 
    CASE WHEN order_count > 1 THEN 'Repeat Customer' ELSE 'One-Time Customer' END AS customer_type,
    COUNT(*) AS count,
    ROUND(100.0 * COUNT(*) / (SELECT COUNT(*) FROM orders_per_customer), 2) AS percentage
FROM orders_per_customer
GROUP BY 1;
2. Logistics, B2B Funnel & Payments
SQL
-- 2A: Logistics & Satisfaction Hypothesis
SELECT 
    review_score,
    ROUND(AVG(julianday(order_delivered_customer_date) - julianday(order_purchase_timestamp)), 1) AS actual_delivery_days,
    ROUND(AVG(julianday(order_delivered_customer_date) - julianday(order_estimated_delivery_date)), 1) AS delay_vs_estimate
FROM orders
JOIN order_reviews USING (order_id)
WHERE order_status = 'delivered' AND order_delivered_customer_date IS NOT NULL
GROUP BY 1 ORDER BY 1 DESC;

-- 2B: B2B Sales Funnel Efficiency
SELECT 
    lq.origin,
    lc.business_segment,
    COUNT(lq.mql_id) AS total_leads,
    COUNT(lc.mql_id) AS closed_deals,
    ROUND(CAST(COUNT(lc.mql_id) AS FLOAT) / COUNT(lq.mql_id) * 100, 2) AS conv_rate
FROM leads_qualified lq
LEFT JOIN leads_closed lc USING (mql_id)
GROUP BY 1, 2 HAVING closed_deals > 0
ORDER BY conv_rate DESC;

-- 2C: Payment & Financing Behavior
SELECT 
    payment_type,
    ROUND(AVG(payment_installments), 1) AS avg_installments,
    SUM(payment_value) AS total_revenue,
    ROUND(100.0 * SUM(payment_value) / (SELECT SUM(payment_value) FROM order_payments), 2) AS rev_share_pct
FROM order_payments
GROUP BY 1 ORDER BY total_revenue DESC;
📈 Strategic Insights (Based on Real Data Results)
1. RFM & Retention: The Loyalty Gap
The Retention Challenge: A massive 97.0% (90,557) of customers purchased only once. Only 3.0% (2,801) are repeat customers.

The Core Asset: 14,781 Champions drive the marketplace with an average spend of 313.16.

The High-Value Leak: 10,933 customers in the "Cant Lose Them" segment have a high historical spend (238.33) but haven't purchased in 473 days.

2. Logistics: The "Retention Killer"
The 5-Star Benchmark: Top-rated orders arrived in 10.7 days, usually 12.7 days BEFORE the estimated deadline.

The 1-Star Experience: Dissatisfied customers waited an average of 21.3 days—twice as long as 5-star customers.

Broken Promises: 1-star reviews correlate with a narrow "safety buffer" of only 3.4 days before the estimate.

3. B2B Sales Funnel Efficiency
Winning Segments: Home Decor leads via organic_search (44 leads) and Car Accessories via paid_search (20 leads) show a 100% conversion rate.

High-Volume Drivers: organic_search is the strongest source for Audio/Electronics and Health/Beauty, consistently converting at 100%.

4. Payment Preferences & Financing
Dominant Method: Credit cards account for 78.34% of total revenue (12.5M), with an average of 3.5 installments.

Cash Alternative: Boleto represents 17.92% of revenue, serving as the primary non-credit option.

🚀 Final Conclusions & Recommendations
Retention Focus: Implement win-back campaigns for the 10,933 "Cant Lose Them" customers. Reactivating this high-spend group is more cost-effective than acquiring new users.

The 11-Day Logistics Goal: To maximize 5-star ratings, Olist should aim for a "Golden Window" of 10.7 days. Deliveries exceeding 14 days significantly damage NPS and repeat purchase probability.

B2B Strategy: Double down on Organic and Paid Search for Health/Beauty and Home Decor, as these show the highest efficiency in closing deals.

🛠️ Tech Stack
SQL Engine: SQLite / PostgreSQL

Key Functions: Window Functions (NTILE), CTEs, Date Math (julianday), Data Aggregation.

Created as a showcase of technical SQL proficiency and data-driven business strategy.
