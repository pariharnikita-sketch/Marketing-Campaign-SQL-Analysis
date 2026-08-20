# Marketing Campaign Performance Analysis (SQL Server / T-SQL)

## 📌 Project Overview
Designed a relational database containing transaction data (`orders`) and budget metrics (`spend`) to analyze digital marketing campaign performance across 4 active channels (GoogleAds, Instagram, Email, Facebook).

## 🛠️ Relational Database Schema & Setup

```sql
-- 1. Orders Table Definition
CREATE TABLE orders (
    order_id INT,
    channel VARCHAR(50),
    campaign VARCHAR(50),
    order_date DATE,
    revenue DECIMAL(10,2)
);

-- 2. Spend Table Definition
CREATE TABLE spend (
    channel VARCHAR(50),
    spend_month VARCHAR(7),
    amount_spent DECIMAL(10,2)
);
```

## 📊 The 7 Core Analytical Queries

### 1. Highest Revenue Generation (GROUP BY / ORDER BY)
**Goal:** Rank total revenue generated per marketing channel.

```sql
SELECT channel, SUM(revenue) AS total_revenue 
FROM orders 
GROUP BY channel 
ORDER BY total_revenue DESC;
```

**Insight:** GoogleAds ($6,400.00) and Instagram ($5,550.00) drove over 80% of total gross revenue.

### 2. Marketing Efficiency / ROAS (Inner JOIN)
**Goal:** Combine table metrics to find Return on Ad Spend (Revenue / Spend).

```sql
SELECT r.channel, ROUND(r.total_revenue / s.total_spent, 2) AS roas
FROM
  (SELECT channel, SUM(revenue) AS total_revenue FROM orders GROUP BY channel) r
JOIN
  (SELECT channel, SUM(amount_spent) AS total_spent FROM spend GROUP BY channel) s
  ON r.channel = s.channel
ORDER BY roas DESC;
```

**Insight:** Instagram was our most efficient channel with a 3.70x ROAS, outperforming GoogleAds (2.91x) on budget efficiency.

### 3. Historical Trends (Date Formatting)
**Goal:** Group and track company revenue growth month-over-month.

```sql
SELECT 
    FORMAT(order_date, 'yyyy-MM') AS revenue_month, 
    SUM(revenue) AS monthly_revenue
FROM orders 
GROUP BY FORMAT(order_date, 'yyyy-MM')
ORDER BY revenue_month ASC;
```

**Insight:** Revenue spiked 38% from January ($3,900) to February ($5,400) before a minor market correction in March ($4,800).

### 4. High-Value Isolation (Subquery)
**Goal:** Extract specific orders that performed higher than the calculated company average.

```sql
SELECT order_id, channel, campaign, revenue 
FROM orders 
WHERE revenue > (SELECT AVG(revenue) FROM orders)
ORDER BY revenue DESC;
```

**Insight:** Successfully isolated 5 elite, high-value transactions driven exclusively by GoogleAds and Instagram.

### 5. Volume Bottlenecks (HAVING Clause)
**Goal:** Isolate failing channels generating critically low volume (fewer than 3 orders).

```sql
SELECT channel, COUNT(*) AS total_orders 
FROM orders 
GROUP BY channel 
HAVING COUNT(*) < 3;
```

**Insight:** Returned 0 rows; all active marketing channels safely met our baseline target order volume.

### 6. Performance Tiering (CASE WHEN)
**Goal:** Classify each marketing channel into High, Medium, or Low tiers based on total Return on Ad Spend (ROAS).

```sql
SELECT r.channel, ROUND(r.total_revenue / s.total_spent, 2) AS roas,
  CASE
    WHEN r.total_revenue / s.total_spent >= 3 THEN 'High'
    WHEN r.total_revenue / s.total_spent >= 1.5 THEN 'Medium'
    ELSE 'Low'
  END AS performance_tier
FROM
  (SELECT channel, SUM(revenue) AS total_revenue FROM orders GROUP BY channel) r
JOIN
  (SELECT channel, SUM(amount_spent) AS total_spent FROM spend GROUP BY channel) s
  ON r.channel = s.channel;
```

**Insight:** Email (6.57x) and Instagram (3.70x) are classified as our elite "High" performance tiers. While GoogleAds brings in the highest raw volume, its efficiency sits in the "Medium" tier (2.91x) alongside Facebook (1.50x).

### 7. Revenue Consistency (Statistics)
**Goal:** Measure how consistent revenue is within each channel using standard deviation.

```sql
SELECT channel, 
  AVG(revenue) AS avg_revenue, 
  STDEV(revenue) AS revenue_stddev
FROM orders
GROUP BY channel;
```

**Insight:** GoogleAds yields the highest average order size ($2,133.33) with relatively controlled variance.

## 📈 Strategic Business Recommendations
- **Budget Optimization:** Shift 15% of underperforming Facebook budget directly into Instagram to maximize macro-efficiency.
- **Scale Engine:** Maintain stable funding for GoogleAds to serve as the high-volume traffic baseline.

---

## Part 2: Power BI Dashboard

Extended the SQL analysis into an interactive Power BI dashboard using the same dataset (orders + spend).

📄 **[View the full dashboard (PDF)](./Marketing%20campaign%20performance%20dashboard.pdf)**

**Visuals included:**
- Bar chart — revenue by channel
- Donut chart — average order value by channel
- Line chart — monthly revenue trend
- Table — campaign-level revenue vs. spend
- KPI cards — total revenue, total spend, average order value
- Slicer — filter by channel
  
**Custom DAX measures:**
Total ROAS = SUM(orders[revenue]) / SUM(spend[amount_spent])
Avg Order Value = AVERAGE(orders[revenue])
