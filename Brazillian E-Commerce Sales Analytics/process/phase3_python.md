# Phase 3 — Python: Exploratory Data Analysis & RFM Segmentation

> **Tool:** Python (pandas, matplotlib, seaborn) in Jupyter Notebook
> **Goal:** Go deeper than SQL — find patterns, correlations, and segment customers using RFM analysis.
> **Output:** `python/eda_analysis.ipynb` — a clean, annotated notebook with markdown commentary

---

## ✅ Step Checklist
- [ ] Jupyter Notebook created and structured with markdown headers
- [ ] All 9 CSVs loaded and merged into master dataframe
- [ ] Analysis 1: Revenue seasonality chart done
- [ ] Analysis 2: Delivery time vs review score visualization done
- [ ] Analysis 3: RFM segmentation completed
- [ ] Analysis 4: Category performance heatmap done
- [ ] Analysis 5: Seller Pareto chart done
- [ ] Clean CSV exported for Power BI
- [ ] All findings documented below
- [ ] GitHub commit pushed

---

## 3.1 — Notebook Setup & Data Loading

**What I did:**
- Created `python/eda_analysis.ipynb` in Jupyter
- Structured the notebook with markdown cells as section headers before writing any code

**Notebook structure (create these markdown headers first):**
```
# E-Commerce Sales Analytics — Exploratory Data Analysis
## 0. Setup & Imports
## 1. Data Loading & Merging
## 2. Revenue Seasonality Analysis
## 3. Delivery Time vs. Customer Satisfaction
## 4. RFM Customer Segmentation
## 5. Category Performance Analysis
## 6. Seller Pareto Analysis
## 7. Key Findings Summary
## 8. Export for Power BI
```

> 💡 Writing headers BEFORE code forces you to think like an analyst, not a coder. Each section should answer one business question.

**Setup cell:**
```python
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import seaborn as sns
import numpy as np
import warnings
warnings.filterwarnings('ignore')

# Style settings
plt.rcParams['figure.figsize'] = (12, 5)
plt.rcParams['axes.spines.top'] = False
plt.rcParams['axes.spines.right'] = False
sns.set_palette('Blues_r')

print("Libraries loaded successfully")
```

**Data loading cell:**
```python
# Load all tables
orders      = pd.read_csv('../data/olist_orders_dataset.csv')
items       = pd.read_csv('../data/olist_order_items_dataset.csv')
payments    = pd.read_csv('../data/olist_order_payments_dataset.csv')
reviews     = pd.read_csv('../data/olist_order_reviews_dataset.csv')
customers   = pd.read_csv('../data/olist_customers_dataset.csv')
sellers     = pd.read_csv('../data/olist_sellers_dataset.csv')
products    = pd.read_csv('../data/olist_products_dataset.csv')
geo         = pd.read_csv('../data/olist_geolocation_dataset.csv')
category_en = pd.read_csv('../data/product_category_name_translation.csv')

# Rename translation columns for clarity
category_en.columns = ['category_portuguese', 'category_english']

print(f"Orders: {len(orders):,} rows")
print(f"Items: {len(items):,} rows")
print(f"Payments: {len(payments):,} rows")
```

**Master dataframe merge:**
```python
# Convert date columns
date_cols = ['order_purchase_timestamp', 'order_delivered_customer_date',
             'order_estimated_delivery_date', 'order_approved_at']
for col in date_cols:
    orders[col] = pd.to_datetime(orders[col])

# Filter to delivered orders only
orders_del = orders[orders['order_status'] == 'delivered'].copy()

# Build master dataframe step by step
df = (orders_del
      .merge(payments[['order_id','payment_value','payment_type']],
             on='order_id', how='left')
      .merge(customers[['customer_id','customer_unique_id','customer_state']],
             on='customer_id', how='left')
      .merge(reviews[['order_id','review_score']].drop_duplicates('order_id'),
             on='order_id', how='left')
)

# Items merged separately (one-to-many)
items_merged = (items
                .merge(products[['product_id','product_category_name']],
                       on='product_id', how='left')
                .merge(category_en, left_on='product_category_name',
                       right_on='category_portuguese', how='left')
)

print(f"Master dataframe: {len(df):,} rows")
print(f"Date range: {df['order_purchase_timestamp'].min().date()} to "
      f"{df['order_purchase_timestamp'].max().date()}")
```

**What I found after loading:**
```
[Paste the printed output here. Write 2-3 sentences about what you notice —
e.g. how many rows in master df, date range, any warnings.]
```

---

## 3.2 — Analysis 1: Revenue Seasonality

**Business question:** When does revenue peak? Is there a seasonal pattern?

**Code:**
```python
## 2. Revenue Seasonality Analysis

# Monthly revenue
df['month'] = df['order_purchase_timestamp'].dt.to_period('M')
monthly_rev = df.groupby('month').agg(
    revenue=('payment_value', 'sum'),
    orders=('order_id', 'count')
).reset_index()
monthly_rev['month_dt'] = monthly_rev['month'].dt.to_timestamp()

# Plot
fig, ax1 = plt.subplots(figsize=(14, 5))
ax2 = ax1.twinx()

ax1.fill_between(monthly_rev['month_dt'], monthly_rev['revenue'],
                 alpha=0.3, color='steelblue')
ax1.plot(monthly_rev['month_dt'], monthly_rev['revenue'],
         color='steelblue', linewidth=2)
ax2.plot(monthly_rev['month_dt'], monthly_rev['orders'],
         color='coral', linewidth=1.5, linestyle='--', label='Order count')

ax1.set_ylabel('Total Revenue (BRL)', color='steelblue')
ax2.set_ylabel('Order Count', color='coral')
ax1.set_xlabel('Month')
plt.title('Monthly Revenue & Order Volume (Delivered Orders Only)')
ax1.yaxis.set_major_formatter(mticker.FuncFormatter(
    lambda x, _: f'R${x:,.0f}'))
plt.tight_layout()
plt.savefig('../screenshots/python_monthly_revenue.png', dpi=150, bbox_inches='tight')
plt.show()

# Print peak month
peak = monthly_rev.loc[monthly_rev['revenue'].idxmax()]
print(f"Peak month: {peak['month']} | Revenue: R${peak['revenue']:,.0f} "
      f"| Orders: {peak['orders']:,}")
```

**What I found:**
```
Peak month: [fill in]
Peak revenue: R$ [fill in]
Peak order count: [fill in]

Pattern observed: [growing steadily? sudden jump? seasonal dip?]
```

**My analysis (write this as a markdown cell in your notebook):**
```
[3-4 sentences. Example: "Revenue shows consistent month-over-month growth from
early 2017 through mid-2018. The November 2017 spike aligns with Black Friday,
suggesting strong promotional sensitivity. A secondary peak in [month] may reflect
[holiday/event]. The last few months of data show [trend], which may be explained by
the dataset being cut off in October 2018."]
```

---

## 3.3 — Analysis 2: Delivery Time vs. Review Score

**Business question:** How strongly does late delivery affect customer satisfaction?

**Code:**
```python
## 3. Delivery Time vs. Customer Satisfaction

# Calculate delivery metrics
df['delivery_days'] = (
    df['order_delivered_customer_date'] -
    df['order_purchase_timestamp']
).dt.days

df['days_vs_estimate'] = (
    df['order_estimated_delivery_date'] -
    df['order_delivered_customer_date']
).dt.days  # positive = delivered early, negative = late

# Remove nulls and outliers for visualization
delivery_df = df[
    df['delivery_days'].notna() &
    df['review_score'].notna() &
    (df['delivery_days'] < 60)
].copy()

# Box plot: delivery days by review score
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))

# Plot 1: delivery days per score
delivery_df.boxplot(column='delivery_days', by='review_score', ax=ax1)
ax1.set_title('Delivery Days by Review Score')
ax1.set_xlabel('Review Score (stars)')
ax1.set_ylabel('Days from Purchase to Delivery')

# Plot 2: avg days early/late per score
avg_timing = delivery_df.groupby('review_score')['days_vs_estimate'].mean()
colors = ['#d73027' if x < 0 else '#1a9850' for x in avg_timing]
avg_timing.plot(kind='bar', color=colors, ax=ax2)
ax2.axhline(0, color='black', linewidth=0.8, linestyle='--')
ax2.set_title('Avg Days Early (green) / Late (red) vs Estimate')
ax2.set_xlabel('Review Score')
ax2.set_ylabel('Days')
ax2.tick_params(axis='x', rotation=0)

plt.suptitle('')
plt.tight_layout()
plt.savefig('../screenshots/python_delivery_satisfaction.png',
            dpi=150, bbox_inches='tight')
plt.show()

# Summary stats
print(delivery_df.groupby('review_score')[
    ['delivery_days','days_vs_estimate']].mean().round(1))
```

**What I found:**
| Review Score | Avg Delivery Days | Avg Days Early/Late |
|---|---|---|
| 5 ⭐ | [fill in] | [fill in] |
| 4 ⭐ | [fill in] | [fill in] |
| 3 ⭐ | [fill in] | [fill in] |
| 2 ⭐ | [fill in] | [fill in] |
| 1 ⭐ | [fill in] | [fill in] |

**My analysis:**
```
[3-4 sentences. This should be your strongest insight.
Example: "The relationship between delivery speed and satisfaction is stark.
5-star orders arrived an average of [X] days early, while 1-star orders arrived
[Y] days late. Every star drop in rating corresponds to roughly [Z] additional
delivery days. This suggests that logistics improvement is the single highest-ROI
action the platform could take to improve customer satisfaction."]
```

---

## 3.4 — Analysis 3: RFM Customer Segmentation

**Business question:** Who are the most valuable customers? Who is at risk of being lost?

**What is RFM?**
- **R**ecency — how recently did they buy? (lower days = better)
- **F**requency — how many times did they buy? (higher = better)
- **M**onetary — how much did they spend in total? (higher = better)

**Code:**
```python
## 4. RFM Customer Segmentation

snapshot_date = df['order_purchase_timestamp'].max() + pd.Timedelta(days=1)

# Calculate RFM values
rfm = df.groupby('customer_unique_id').agg(
    Recency=('order_purchase_timestamp',
             lambda x: (snapshot_date - x.max()).days),
    Frequency=('order_id', 'count'),
    Monetary=('payment_value', 'sum')
).reset_index()

# Score each dimension 1-4 (4 = best)
rfm['R_score'] = pd.qcut(rfm['Recency'],  q=4,
                          labels=[4,3,2,1])  # reversed: low recency = recent = good
rfm['F_score'] = pd.qcut(rfm['Frequency'].rank(method='first'),
                          q=4, labels=[1,2,3,4])
rfm['M_score'] = pd.qcut(rfm['Monetary'],  q=4, labels=[1,2,3,4])

rfm['RFM_score'] = (rfm['R_score'].astype(str) +
                    rfm['F_score'].astype(str) +
                    rfm['M_score'].astype(str))

# Assign segments
def assign_segment(row):
    r, f, m = int(row['R_score']), int(row['F_score']), int(row['M_score'])
    if r >= 4 and f >= 3 and m >= 3:
        return 'Champion'
    elif r >= 3 and f >= 2:
        return 'Loyal'
    elif r >= 4 and f <= 2:
        return 'New Customer'
    elif r >= 3 and f <= 2:
        return 'Promising'
    elif r <= 2 and f >= 3:
        return 'At Risk'
    elif r <= 2 and f <= 2 and m >= 3:
        return 'Cannot Lose'
    else:
        return 'Lost'

rfm['Segment'] = rfm.apply(assign_segment, axis=1)

# Summary table
segment_summary = rfm.groupby('Segment').agg(
    Count=('customer_unique_id', 'count'),
    Avg_Recency=('Recency', 'mean'),
    Avg_Frequency=('Frequency', 'mean'),
    Avg_Monetary=('Monetary', 'mean')
).round(1).sort_values('Count', ascending=False)

print(segment_summary)

# Save for Power BI
rfm.to_csv('../data/rfm_segments.csv', index=False)
print("RFM segments saved to /data/rfm_segments.csv")
```

**What I found:**
| Segment | Count | % of Customers | Avg Spend | My Notes |
|---|---|---|---|---|
| Champion | [fill in] | [fill in]% | [fill in] | [what makes them special?] |
| Loyal | [fill in] | [fill in]% | [fill in] | |
| New Customer | [fill in] | [fill in]% | [fill in] | |
| Promising | [fill in] | [fill in]% | [fill in] | |
| At Risk | [fill in] | [fill in]% | [fill in] | [what should the business do?] |
| Cannot Lose | [fill in] | [fill in]% | [fill in] | |
| Lost | [fill in] | [fill in]% | [fill in] | |

**My analysis:**
```
[4-5 sentences. This section should connect directly to your recommendation.
Example: "Champions represent only [X]% of customers but likely account for a
disproportionate share of revenue. The At Risk segment is particularly concerning —
these are customers who used to buy frequently but haven't returned in [X] days.
The Lost segment ([X]% of all customers) represents a large base that the platform
has failed to retain. A targeted re-engagement campaign for the At Risk and
Cannot Lose segments could recover significant revenue without acquiring new customers."]
```

---

## 3.5 — Analysis 4: Category Performance Heatmap

**Business question:** Which categories are high-revenue AND high-satisfaction?

**Code:**
```python
## 5. Category Performance Analysis

# Merge items with orders and reviews
cat_df = (items_merged
          .merge(orders_del[['order_id','order_purchase_timestamp']],
                 on='order_id', how='left')
          .merge(reviews[['order_id','review_score']].drop_duplicates('order_id'),
                 on='order_id', how='left')
)

cat_df['category'] = cat_df['category_english'].fillna(
    cat_df['product_category_name']).fillna('Unknown')

# Category summary
cat_summary = cat_df.groupby('category').agg(
    revenue=('price', 'sum'),
    units_sold=('order_item_id', 'count'),
    avg_price=('price', 'mean'),
    avg_review=('review_score', 'mean')
).reset_index()

cat_summary = cat_summary[cat_summary['units_sold'] >= 100]  # filter low-volume

# Top 15 by revenue
top15 = cat_summary.nlargest(15, 'revenue')

# Heatmap of normalized metrics
heatmap_data = top15[['revenue','units_sold','avg_price','avg_review']].copy()
heatmap_norm = (heatmap_data - heatmap_data.min()) / (
    heatmap_data.max() - heatmap_data.min())
heatmap_norm.index = top15['category'].str.replace('_', ' ').str.title()

fig, ax = plt.subplots(figsize=(10, 8))
sns.heatmap(heatmap_norm, annot=heatmap_data.round(0), fmt='g',
            cmap='YlOrRd', ax=ax, linewidths=0.5)
ax.set_title('Top 15 Categories — Normalized Performance Metrics')
plt.tight_layout()
plt.savefig('../screenshots/python_category_heatmap.png',
            dpi=150, bbox_inches='tight')
plt.show()
```

**What I found:**
```
[Write 2-3 observations from the heatmap.
Example: "Health & Beauty ranks high on both revenue and avg_review,
making it a strong category to prioritize. Computers & Accessories
generates high revenue but has below-average review scores,
suggesting product quality or delivery issues in this category."]
```

---

## 3.6 — Analysis 5: Seller Pareto (80/20 Rule)

**Business question:** Do 20% of sellers generate 80% of revenue?

**Code:**
```python
## 6. Seller Pareto Analysis

seller_rev = (items_merged
              .merge(orders_del[['order_id']], on='order_id', how='inner')
              .groupby('seller_id')['price'].sum()
              .reset_index()
              .rename(columns={'price': 'revenue'})
              .sort_values('revenue', ascending=False)
)

seller_rev['cumulative_pct'] = (seller_rev['revenue'].cumsum() /
                                 seller_rev['revenue'].sum() * 100)
seller_rev['seller_pct'] = (range(1, len(seller_rev)+1)) / len(seller_rev) * 100

# Find 80% threshold
threshold_80 = seller_rev[seller_rev['cumulative_pct'] <= 80].shape[0]
pct_sellers  = threshold_80 / len(seller_rev) * 100

fig, ax = plt.subplots(figsize=(12, 5))
ax.plot(seller_rev['seller_pct'], seller_rev['cumulative_pct'],
        color='steelblue', linewidth=2)
ax.axhline(80, color='red', linestyle='--', linewidth=1,
           label='80% revenue threshold')
ax.axvline(pct_sellers, color='orange', linestyle='--', linewidth=1,
           label=f'{pct_sellers:.1f}% of sellers')
ax.fill_between(seller_rev['seller_pct'], seller_rev['cumulative_pct'],
                alpha=0.1, color='steelblue')
ax.set_xlabel('Cumulative % of Sellers')
ax.set_ylabel('Cumulative % of Revenue')
ax.set_title('Seller Revenue Concentration (Pareto Curve)')
ax.legend()
plt.tight_layout()
plt.savefig('../screenshots/python_seller_pareto.png',
            dpi=150, bbox_inches='tight')
plt.show()

print(f"Top {pct_sellers:.1f}% of sellers generate 80% of revenue")
print(f"({threshold_80} sellers out of {len(seller_rev):,} total)")
```

**What I found:**
```
Top [fill in]% of sellers generate 80% of revenue
([fill in] sellers out of [fill in] total)

My analysis: [Write 2-3 sentences about what this means.
Does it confirm or challenge the Pareto rule?
What should the platform do about the long tail of low-revenue sellers?]
```

---

## 3.7 — Export Clean Data for Power BI

**What I did:**
```python
## 8. Export for Power BI

# Master orders dataset
df['delivery_days'] = (
    df['order_delivered_customer_date'] -
    df['order_purchase_timestamp']
).dt.days

df['is_late'] = (
    df['order_delivered_customer_date'] >
    df['order_estimated_delivery_date']
).astype(int)

df['year_month'] = df['order_purchase_timestamp'].dt.to_period('M').astype(str)

export_cols = [
    'order_id','customer_unique_id','customer_state',
    'order_purchase_timestamp','year_month',
    'payment_value','payment_type',
    'review_score','delivery_days','is_late'
]

df[export_cols].to_csv('../data/cleaned_orders.csv', index=False)

# Category summary
cat_summary.to_csv('../data/category_summary.csv', index=False)

# RFM already saved in 3.4

print("Exported:")
print(f"  cleaned_orders.csv     — {len(df):,} rows")
print(f"  category_summary.csv   — {len(cat_summary):,} rows")
print(f"  rfm_segments.csv       — {len(rfm):,} rows")
```

**Files exported to `/data`:**
| File | Rows | Used For |
|---|---|---|
| cleaned_orders.csv | [fill in] | Main Power BI fact table |
| category_summary.csv | [fill in] | Category page visuals |
| rfm_segments.csv | [fill in] | Customer segment table |

---

## 3.8 — Summary: What I Learned in Phase 3

**5 most important findings from Python EDA:**
```
1. Seasonality: [your finding]
2. Delivery impact: [your finding — be specific with numbers]
3. RFM insight: [which segment is most actionable?]
4. Category insight: [which category surprised you?]
5. Seller concentration: [your Pareto finding]
```

**Python skills I used in this phase:**
```
- pandas merging (multi-table joins using .merge())
- datetime manipulation (dt accessor, Timedelta)
- GroupBy aggregations with custom lambda functions
- pd.qcut for quantile-based scoring
- matplotlib dual-axis charts
- seaborn heatmap with normalized data
- Data export to CSV for downstream tools
```

**How these findings will shape my Power BI dashboard:**
```
[Write 3-4 sentences connecting your findings to specific visuals.
Example: "The delivery-satisfaction finding is so clear it deserves its own
dashboard page. I will build a scatter plot of delivery days vs review score
and annotate it with the key numbers. The RFM segments will appear as a
donut chart on the customer page with a table showing each segment's size and value."]
```

---

## 3.9 — GitHub Commit

```bash
git add python/eda_analysis.ipynb
git add data/cleaned_orders.csv
git add data/rfm_segments.csv
git add data/category_summary.csv
git add screenshots/python_*.png
git add process/phase3_python.md
git commit -m "Phase 3 complete: Python EDA — seasonality, delivery analysis, RFM segmentation, Pareto"
git push origin main
```

---

*Previous: [Phase 2 — SQL](./phase2_sql.md)*
*Next: [Phase 4 — Power BI](./phase4_powerbi.md)*
