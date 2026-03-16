# Phase 1 — Excel: Data Inspection & Cleaning

> **Tool:** Microsoft Excel
> **Goal:** Understand the raw data before writing a single line of code. Document every issue you find.
> **Output:** `excel/data_issues_log.xlsx` with 3 sheets completed

---

## ✅ Step Checklist
- [ ] All 9 CSVs downloaded and saved in `/data`
- [ ] `data_issues_log.xlsx` created in `/excel`
- [ ] Sheet 1: Issues Log filled in
- [ ] Sheet 2: Row Counts filled in
- [ ] Sheet 3: Data Dictionary filled in
- [ ] Pivot table sanity check done
- [ ] Observations written below
- [ ] GitHub commit pushed

---

## 1.1 — Dataset Download

**What I did:**
- Downloaded the Olist Brazilian E-Commerce Public Dataset from Kaggle
- Link: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
- Unzipped and placed all 9 CSV files into `/data`

**Files received:**
| # | File Name | Description |
|---|-----------|-------------|
| 1 | olist_orders_dataset.csv | Main fact table — one row per order |
| 2 | olist_order_items_dataset.csv | Items within each order |
| 3 | olist_order_payments_dataset.csv | Payment details per order |
| 4 | olist_order_reviews_dataset.csv | Customer review scores |
| 5 | olist_customers_dataset.csv | Customer info |
| 6 | olist_sellers_dataset.csv | Seller info |
| 7 | olist_products_dataset.csv | Product details |
| 8 | olist_geolocation_dataset.csv | ZIP code coordinates |
| 9 | product_category_name_translation.csv | Portuguese → English category names |

---

## 1.2 — Row Count Validation

**What I did:**
- Opened each CSV in Excel
- Used `=COUNTA(A:A)-1` to count rows (subtract 1 for header)
- Recorded results in Sheet 2 of `data_issues_log.xlsx`

**What I found:**
| File | Row Count | My Notes |
|------|-----------|----------|
| olist_orders_dataset.csv | 99,441 | Main fact table |
| olist_order_items_dataset.csv | 112,650 | More rows than orders — multiple items per order |
| olist_order_payments_dataset.csv | 103,886 | More rows than orders — some split payments |
| olist_order_reviews_dataset.csv | 99,224 | ~217 orders have no review — normal |
| olist_customers_dataset.csv | 99,441 | Matches orders exactly — clean 1-to-1 |
| olist_sellers_dataset.csv | 3,095 | ~32 orders per seller on average |
| olist_products_dataset.csv | 32,951 | 32,951 unique products |
| olist_geolocation_dataset.csv | 1,000,163 | Too large, geography not relevant — will exclude |
| product_category_name_translation.csv | 71 | Small lookup table, 71 categories |

**My key observation from row counts:**
```
- order_items has 112,650 rows vs 99,441 orders, meaning some orders contain multiple items. I must GROUP BY order_id when calculating revenue to avoid double-counting. 
- order_payments also has more rows than orders (103,886) for the same reason — some customers split payments across two methods.
- customers matches orders exactly at 99,441 — confirming a clean 1-to-1 relationship.
```

---

## 1.3 — Missing Value Check

**What I did:**
- For each key column, used `=COUNTBLANK(column)` to find nulls
- Focused on columns I will use in analysis (not every column)
- Recorded findings in Sheet 1 of `data_issues_log.xlsx`

**What I found:**
| # | File | Column | Null Count | My Decision |
|---|------|--------|------------|-------------|
| 1 | olist_orders | order_approved_at | ~160 | [write your decision] |
| 2 | olist_orders | order_delivered_carrier_date | ~1,783 | [write your decision] |
| 3 | olist_orders | order_delivered_customer_date | ~2,965 | [write your decision] |
| 4 | olist_orders | order_estimated_delivery_date | 0 | [write your observation] |
| 5 | olist_products | product_category_name | ~610 | [write your decision] |
| 6 | olist_order_reviews | review_comment_message | ~58,247 | [write your decision] |

> 💡 **Guided observation:** Most nulls in order dates are from orders that were never delivered (cancelled/unavailable). This is NOT bad data — it is meaningful. Write that you will filter to `order_status = 'delivered'` in SQL to handle this cleanly.

**My decision on nulls:**
```
[Write your decisions here. Example: "Null delivery dates exist because ~3,000 orders
were cancelled or never shipped. I will filter to delivered orders only in SQL analysis.
Null review comments are fine — not all customers leave written feedback."]
```

---

## 1.4 — Order Status Distribution (Pivot Table)

**What I did:**
- Opened `olist_orders_dataset.csv` in Excel
- Selected all data → Insert → PivotTable
- Rows: `order_status` | Values: `order_id` (Count)

**What I found:**
| order_status | Count | % of Total |
|---|---|---|
| delivered | [fill in] | [fill in]% |
| shipped | [fill in] | [fill in]% |
| cancelled | [fill in] | [fill in]% |
| unavailable | [fill in] | [fill in]% |
| invoiced | [fill in] | [fill in]% |
| processing | [fill in] | [fill in]% |
| created | [fill in] | [fill in]% |
| approved | [fill in] | [fill in]% |

> 💡 **Guided observation:** "delivered" should be ~97% of all orders. This is the only status where we have complete data (purchase date, payment, delivery date, review). Write that all revenue and delivery analysis will be filtered to delivered orders.

**My key observation:**
```
[Write here. Example: "96,478 out of 99,441 orders (97%) are 'delivered'. All revenue,
delivery time, and satisfaction analysis will filter to delivered orders only to ensure
data completeness and consistency."]
```

---

## 1.5 — Date Range Check

**What I did:**
- In `olist_orders_dataset.csv`, used `=MIN(column)` and `=MAX(column)` on `order_purchase_timestamp`

**What I found:**
| | Value |
|---|---|
| Earliest order | [fill in — expected: around Sep 2016] |
| Latest order | [fill in — expected: around Oct 2018] |
| Total date span | [calculate: how many months?] |

> 💡 **Guided observation:** The dataset covers about 25 months. Note whether any months seem unusually low in volume — this could indicate data collection started mid-month or the dataset was cut off.

**My observation:**
```
[Write here. Example: "Data runs from Sep 2016 to Oct 2018 — about 25 months.
The first and last months may have incomplete data as the platform was
ramping up / data collection was cut off. I will note this in the dashboard."]
```

---

## 1.6 — Data Dictionary (Sheet 3)

**What I did:**
- Documented all key columns I will use in analysis
- Added plain-English descriptions for each

**Key columns documented:**

| Table | Column | Type | Description | Used For |
|-------|--------|------|-------------|----------|
| orders | order_id | STRING | Unique order ID | Joining all tables |
| orders | customer_id | STRING | Links to customers table | Customer analysis |
| orders | order_status | STRING | delivered/cancelled/etc | Filtering |
| orders | order_purchase_timestamp | DATETIME | When order was placed | Revenue trends |
| orders | order_delivered_customer_date | DATETIME | When customer received order | Delivery time |
| orders | order_estimated_delivery_date | DATETIME | Promised delivery date | Late delivery flag |
| order_items | order_id | STRING | Links to orders | Revenue calculation |
| order_items | product_id | STRING | Links to products | Category analysis |
| order_items | seller_id | STRING | Links to sellers | Seller performance |
| order_items | price | FLOAT | Item price in BRL | Revenue |
| order_items | freight_value | FLOAT | Shipping cost | Cost analysis |
| order_payments | payment_value | FLOAT | Total payment | Revenue (incl. freight) |
| order_payments | payment_type | STRING | credit_card/boleto/etc | Payment analysis |
| order_reviews | review_score | INT | 1-5 star rating | Satisfaction analysis |
| customers | customer_unique_id | STRING | True unique customer ID | Repeat purchase rate |
| customers | customer_state | STRING | Brazilian state | Geographic analysis |
| products | product_category_name | STRING | Category in Portuguese | Category analysis |
| sellers | seller_state | STRING | Seller location | Seller analysis |

> 💡 **Important note to write:** `customer_id` and `customer_unique_id` are DIFFERENT. `customer_id` is tied to a specific order — the same person can have multiple `customer_id` values. `customer_unique_id` is the true unique person. Always use `customer_unique_id` for repeat customer analysis.

**My note on customer_id vs customer_unique_id:**
```
[Write this in your own words — it will come up in interviews]
```

---

## 1.7 — Summary: What I Learned in Phase 1

> Fill this in after completing all steps above. This is the most important section — it shows analytical thinking.

**3 most important data quality findings:**
```
1. [Your finding + what it means for analysis]
2. [Your finding + what it means for analysis]
3. [Your finding + what it means for analysis]
```

**3 decisions I made about the data:**
```
1. [Decision + reason. Example: "I will filter all analysis to delivered orders because
   other statuses have incomplete date and payment data."]
2. [Decision + reason]
3. [Decision + reason]
```

**What surprised me:**
```
[Write 1-2 things that were unexpected. Interviewers love this question —
having a genuine answer here is valuable.]
```

---

## 1.8 — GitHub Commit

**What I did:**
```bash
git add excel/data_issues_log.xlsx
git add process/phase1_excel.md
git commit -m "Phase 1 complete: Excel data inspection, issues log, data dictionary"
git push origin main
```

**Commit message explains:** what was done, which phase, which files

---

*Next: [Phase 2 — SQL](./phase2_sql.md)*
