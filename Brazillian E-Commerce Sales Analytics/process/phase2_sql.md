# Phase 2 — SQL: Querying & Exploration

> **Tool:** SQLite + DB Browser for SQLite
> **Goal:** Answer structured business questions using SQL. Demonstrate JOIN, aggregation, window function, and CTE skills.
> **Output:** 4 `.sql` files saved in `/sql`, findings documented below

---

## ✅ Step Checklist
- [ ] All CSVs imported into SQLite database
- [ ] Database schema verified (all tables present)
- [ ] Query 1: Monthly revenue trend written and run
- [ ] Query 2: Top product categories written and run
- [ ] Query 3: Customer repeat rate written and run
- [ ] Query 4: Delivery time vs review score written and run
- [ ] All 4 .sql files saved in `/sql`
- [ ] Findings documented below
- [ ] GitHub commit pushed

---

## 2.1 — Setting Up SQLite

**What I did:**
- Opened DB Browser for SQLite
- Created a new database: `File → New Database → save as olist.db inside /data`
- Imported each CSV: `File → Import → Table from CSV`

**Import order (important — import in this sequence):**
```
1. olist_customers_dataset.csv         → table name: customers
2. olist_sellers_dataset.csv           → table name: sellers
3. olist_products_dataset.csv          → table name: products
4. product_category_name_translation.csv → table name: category_translation
5. olist_orders_dataset.csv            → table name: orders
6. olist_order_items_dataset.csv       → table name: order_items
7. olist_order_payments_dataset.csv    → table name: order_payments
8. olist_order_reviews_dataset.csv     → table name: order_reviews
9. olist_geolocation_dataset.csv       → table name: geolocation
```

> 💡 Import parent tables before child tables to avoid confusion. Orders must exist before order_items, payments, and reviews.

**Verification query — run this first:**
```sql
SELECT name FROM sqlite_master WHERE type='table';
```

**What I expected to see:**
```
customers, sellers, products, category_translation,
orders, order_items, order_payments, order_reviews, geolocation
```

**What I actually saw:**
```
[paste your output here]
```

---

## 2.2 — Query 1: Monthly Revenue Trend

**Business question:** How has revenue changed month by month? Are there seasonal peaks?

**File:** `sql/01_monthly_revenue.sql`

**The query:**
```sql
SELECT
    strftime('%Y-%m', o.order_purchase_timestamp) AS month,
    ROUND(SUM(p.payment_value), 2)                AS total_revenue,
    COUNT(DISTINCT o.order_id)                    AS total_orders,
    ROUND(AVG(p.payment_value), 2)                AS avg_order_value
FROM orders o
JOIN order_payments p ON o.order_id = p.order_id
WHERE o.order_status = 'delivered'
GROUP BY month
ORDER BY month;
```

**SQL skills demonstrated:** JOIN, aggregate functions, GROUP BY, WHERE filter, date formatting

**What I found:**
| Observation | My Notes |
|---|---|
| Peak month | [fill in — expected: Nov 2017] |
| Lowest month | [fill in — expected: early months when platform was new] |
| Overall trend | [growing / stable / declining?] |
| Biggest single month jump | [from which month to which?] |

**My analysis of the results:**
```
[Write 3-4 sentences. Example: "Revenue grew consistently from late 2016 through
mid-2018. The peak month was November 2017, likely driven by Black Friday promotions.
There is a noticeable dip in [month] which may reflect [reason]. Average order value
stayed relatively stable at around R$[X], suggesting growth was driven by order volume
rather than higher-value purchases."]
```

**What this means for the dashboard:**
```
[Write 1-2 sentences on what visual you'll build from this.
Example: "I will build a line chart of monthly revenue with a trend line.
I will annotate the November 2017 peak with a label explaining Black Friday."]
```

---

## 2.3 — Query 2: Top Product Categories

**Business question:** Which product categories drive the most revenue and volume?

**File:** `sql/02_top_categories.sql`

**The query:**
```sql
SELECT
    COALESCE(ct.string_field1, p.product_category_name, 'Unknown') AS category_english,
    ROUND(SUM(oi.price), 2)          AS total_revenue,
    COUNT(oi.order_item_id)          AS units_sold,
    ROUND(AVG(oi.price), 2)          AS avg_price,
    COUNT(DISTINCT oi.order_id)      AS total_orders
FROM order_items oi
JOIN products p         ON oi.product_id   = p.product_id
JOIN orders o           ON oi.order_id     = o.order_id
LEFT JOIN category_translation ct ON p.product_category_name = ct.string_field0
WHERE o.order_status = 'delivered'
GROUP BY category_english
ORDER BY total_revenue DESC
LIMIT 15;
```

**SQL skills demonstrated:** JOIN, LEFT JOIN, COALESCE for null handling, multiple aggregations, aliasing

**What I found:**
| Rank | Category | Revenue (BRL) | Units Sold | Avg Price | My Notes |
|------|----------|---------------|------------|-----------|----------|
| 1 | [fill in] | [fill in] | [fill in] | [fill in] | |
| 2 | [fill in] | [fill in] | [fill in] | [fill in] | |
| 3 | [fill in] | [fill in] | [fill in] | [fill in] | |
| 4 | [fill in] | [fill in] | [fill in] | [fill in] | |
| 5 | [fill in] | [fill in] | [fill in] | [fill in] | |

> 💡 **Guided observation:** Look for categories where avg_price is high but units_sold is low — these are high-ticket items. Compare to categories with low avg_price but high volume. Both can generate similar revenue through different strategies. Write this pattern down.

**My analysis:**
```
[Write 3-4 sentences. Example: "The top category by revenue is [X], which accounts
for approximately [Y]% of total revenue. Interestingly, [category] ranks high in
revenue despite a lower unit count, indicating it is a premium-priced category.
[Category] leads in units sold but has a lower average price — a high-volume,
low-margin segment."]
```

---

## 2.4 — Query 3: Customer Repeat Purchase Rate

**Business question:** What percentage of customers made more than one purchase? (Customer loyalty signal)

**File:** `sql/03_repeat_rate.sql`

**The query:**
```sql
-- Step 1: Count orders per unique customer
WITH customer_orders AS (
    SELECT
        c.customer_unique_id,
        COUNT(o.order_id) AS num_orders
    FROM orders o
    JOIN customers c ON o.customer_id = c.customer_id
    WHERE o.order_status = 'delivered'
    GROUP BY c.customer_unique_id
),
-- Step 2: Classify as repeat or one-time
classified AS (
    SELECT
        customer_unique_id,
        num_orders,
        CASE WHEN num_orders > 1 THEN 'Repeat' ELSE 'One-time' END AS customer_type
    FROM customer_orders
)
-- Step 3: Calculate rates
SELECT
    customer_type,
    COUNT(*)                                        AS customer_count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2) AS percentage
FROM classified
GROUP BY customer_type;
```

**SQL skills demonstrated:** CTE (WITH clause), CASE WHEN, window function (SUM OVER()), subquery layering

**What I found:**
| Customer Type | Count | Percentage |
|---|---|---|
| One-time | [fill in] | [fill in]% |
| Repeat | [fill in] | [fill in]% |

**My analysis:**
```
[Write 3-4 sentences. Example: "Only [X]% of customers made more than one purchase,
meaning the vast majority of customers are one-time buyers. This is a significant
business insight — the platform appears to struggle with customer retention.
For context, a healthy e-commerce retention rate is typically 20-30%.
This finding directly motivates the RFM segmentation I will do in Python —
understanding WHO the repeat customers are could reveal strategies to increase loyalty."]
```

> 💡 **This is a powerful insight for your story.** A low repeat rate is not necessarily bad data — it is a real business problem that your dashboard can highlight. It makes your recommendation section much stronger.

---

## 2.5 — Query 4: Delivery Time vs. Review Score

**Business question:** Does late delivery hurt customer satisfaction? By how much?

**File:** `sql/04_delivery_vs_rating.sql`

**The query:**
```sql
SELECT
    r.review_score,
    COUNT(*)                                                              AS order_count,
    ROUND(AVG(
        julianday(o.order_delivered_customer_date) -
        julianday(o.order_purchase_timestamp)
    ), 1)                                                                 AS avg_delivery_days,
    ROUND(AVG(
        julianday(o.order_estimated_delivery_date) -
        julianday(o.order_delivered_customer_date)
    ), 1)                                                                 AS avg_days_early_late
FROM orders o
JOIN order_reviews r ON o.order_id = r.order_id
WHERE o.order_status = 'delivered'
  AND o.order_delivered_customer_date IS NOT NULL
GROUP BY r.review_score
ORDER BY r.review_score DESC;
```

**SQL skills demonstrated:** julianday() for date arithmetic, multi-table JOIN, conditional aggregation, NULL filtering

> 💡 **Reading avg_days_early_late:** A positive number means delivered BEFORE the estimated date (good). A negative number means delivered AFTER (late). This is a key metric.

**What I found:**
| Review Score | Order Count | Avg Delivery Days | Avg Days Early/Late | My Notes |
|---|---|---|---|---|
| 5 ⭐ | [fill in] | [fill in] | [fill in] | [on time?] |
| 4 ⭐ | [fill in] | [fill in] | [fill in] | |
| 3 ⭐ | [fill in] | [fill in] | [fill in] | |
| 2 ⭐ | [fill in] | [fill in] | [fill in] | |
| 1 ⭐ | [fill in] | [fill in] | [fill in] | [very late?] |

**My analysis:**
```
[Write 3-4 sentences. Example: "The results show a clear pattern: 5-star orders were
delivered on average [X] days early, while 1-star orders were delivered [Y] days late.
Average delivery time nearly doubles from [X] days for 5-star to [Y] days for 1-star reviews.
This is a strong causal signal — late delivery is the primary driver of negative reviews.
The practical implication is that improving logistics in high-delay regions could
significantly lift overall platform satisfaction scores."]
```

---

## 2.6 — Bonus Query: Seller Performance (Optional but recommended)

**Business question:** Which sellers generate the most revenue, and how concentrated is revenue among top sellers?

**File:** `sql/05_seller_performance.sql`

```sql
SELECT
    oi.seller_id,
    s.seller_state,
    ROUND(SUM(oi.price), 2)          AS total_revenue,
    COUNT(DISTINCT oi.order_id)      AS total_orders,
    ROUND(AVG(r.review_score), 2)    AS avg_review_score,
    ROUND(AVG(oi.price), 2)          AS avg_item_price
FROM order_items oi
JOIN orders o      ON oi.order_id   = o.order_id
JOIN sellers s     ON oi.seller_id  = s.seller_id
JOIN order_reviews r ON o.order_id  = r.order_id
WHERE o.order_status = 'delivered'
GROUP BY oi.seller_id, s.seller_state
ORDER BY total_revenue DESC
LIMIT 20;
```

**What I found:**
```
[Write observations. Look for: Do top sellers also have high review scores?
Is revenue concentrated in a few sellers (Pareto effect)?
Are top sellers from the same state?]
```

---

## 2.7 — Summary: What I Learned in Phase 2

**Most important SQL findings:**
```
1. Revenue trend: [your finding — peak month, growth pattern]
2. Top category: [which category dominates and why it matters]
3. Repeat rate: [the % and what it signals about the business]
4. Delivery insight: [how many extra days for 1-star vs 5-star orders]
```

**SQL techniques I used in this phase:**
```
- JOINs (INNER and LEFT JOIN): [explain where and why]
- Aggregations (SUM, COUNT, AVG): [explain what you calculated]
- CTEs (WITH clause): [explain which query used it and why it was cleaner]
- Window functions (OVER()): [explain what it calculated]
- Date arithmetic (julianday): [explain what it measured]
- COALESCE: [explain where you used it for null handling]
```

**What surprised me:**
```
[Write 1-2 genuine surprises from the data. These are excellent interview talking points.]
```

**What questions do I still have that Python can answer?**
```
[Write 2-3 questions. Example:
"1. Are there seasonal patterns within months, not just between months?
 2. Which specific customer segments are most likely to repeat purchase?
 3. Is the delivery problem worse in certain regions?"]
```

---

## 2.8 — GitHub Commit

```bash
git add sql/
git add process/phase2_sql.md
git commit -m "Phase 2 complete: 4 SQL queries — revenue trend, categories, repeat rate, delivery analysis"
git push origin main
```

---

*Previous: [Phase 1 — Excel](./phase1_excel.md)*
*Next: [Phase 3 — Python](./phase3_python.md)*
