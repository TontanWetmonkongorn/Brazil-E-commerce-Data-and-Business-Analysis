# Phase 4 — Power BI: Dashboard Design & Storytelling

> **Tool:** Power BI Desktop
> **Goal:** Build a 3-page interactive dashboard that tells one clear business story — not a data dump.
> **Output:** `powerbi/ecommerce_dashboard.pbix` + screenshots in `/screenshots`

---

## ✅ Step Checklist
- [ ] All 3 CSV files loaded into Power BI
- [ ] Relationships set up in Model view
- [ ] Date table created
- [ ] All DAX measures written
- [ ] Page 1: Executive Overview complete
- [ ] Page 2: Customer & Delivery Analysis complete
- [ ] Page 3: Seller & Category Performance complete
- [ ] Color theme applied consistently
- [ ] "So what?" text boxes added to each page
- [ ] Screenshots saved to `/screenshots`
- [ ] GitHub commit pushed

---

## 4.1 — Load Data into Power BI

**What I did:**
- Opened Power BI Desktop
- Home → Get Data → Text/CSV
- Loaded these 3 files from `/data`:

| File | Loaded As | Purpose |
|---|---|---|
| cleaned_orders.csv | orders | Main fact table |
| rfm_segments.csv | rfm | Customer segments |
| category_summary.csv | categories | Category metrics |

**Steps for each file:**
1. Home → Get Data → Text/CSV
2. Navigate to `/data` folder
3. Select file → Load (do NOT click Transform yet)
4. Repeat for all 3 files

**What I saw after loading:**
```
[Write the row counts Power BI shows for each table — should match your Python exports]
```

---

## 4.2 — Set Up Relationships (Model View)

**What I did:**
- Clicked the **Model** icon (left sidebar, looks like a diagram)
- Connected tables via drag-and-drop

**Relationships to create:**
| From Table | From Column | To Table | To Column | Cardinality |
|---|---|---|---|---|
| orders | customer_unique_id | rfm | customer_unique_id | Many-to-One |

> 💡 The `categories` table stands alone — it's not related to orders directly. You'll use it independently on Page 3.

**What I verified:**
```
[Write: "I confirmed both relationships show the correct cardinality arrow
in Model view. The orders → rfm relationship shows a many-to-one (*→1) arrow."]
```

---

## 4.3 — Create a Date Table

**Why:** Power BI time intelligence (MoM, YoY calculations) requires a proper date table.

**What I did:**
- Home → Enter Data → (this opens a blank table)
- Instead, used DAX: Modeling → New Table

**DAX to create date table:**
```dax
DateTable =
CALENDAR(
    DATE(2016, 9, 1),
    DATE(2018, 10, 31)
)
```

**Then add columns to the date table (New Column for each):**
```dax
Year       = YEAR(DateTable[Date])
Month      = MONTH(DateTable[Date])
MonthName  = FORMAT(DateTable[Date], "MMM YYYY")
Quarter    = "Q" & QUARTER(DateTable[Date])
YearMonth  = FORMAT(DateTable[Date], "YYYY-MM")
```

**Connect date table to orders:**
- Model view → drag `DateTable[Date]` to `orders[order_purchase_timestamp]`
- Cardinality: One-to-Many (1→*)

**What I verified:**
```
[Write: "Date table connected successfully. In Model view, the relationship
arrow points from DateTable (1) to orders (*)."]
```

---

## 4.4 — Write DAX Measures

**What I did:**
- Selected the `orders` table
- Home → New Measure (wrote each measure below)
- Organized all measures into a dedicated "Measures" table

**Create a measures table (best practice):**
- Enter Data → create a blank table called `_Measures` with one column "placeholder"
- Store all your measures here to keep the model clean

**Core measures to write:**

```dax
-- Total Revenue
Total Revenue =
SUM(orders[payment_value])

-- Total Orders
Total Orders =
COUNTROWS(orders)

-- Average Order Value
Avg Order Value =
DIVIDE([Total Revenue], [Total Orders])

-- Repeat Customer Rate
Repeat Customer Rate =
DIVIDE(
    CALCULATE(
        DISTINCTCOUNT(orders[customer_unique_id]),
        FILTER(rfm, rfm[Frequency] > 1)
    ),
    DISTINCTCOUNT(orders[customer_unique_id])
)

-- Average Review Score
Avg Review Score =
AVERAGE(orders[review_score])

-- Average Delivery Days
Avg Delivery Days =
AVERAGE(orders[delivery_days])

-- Late Delivery Rate
Late Delivery Rate =
DIVIDE(
    CALCULATE(COUNTROWS(orders), orders[is_late] = 1),
    COUNTROWS(orders)
)

-- Month-over-Month Revenue Growth
MoM Growth =
VAR current_month = [Total Revenue]
VAR prev_month =
    CALCULATE([Total Revenue],
        DATEADD(DateTable[Date], -1, MONTH))
RETURN
    DIVIDE(current_month - prev_month, prev_month)
```

**Measures I wrote successfully:**
```
[Tick off each measure as you write it and confirm it shows a value
in the report view. Write any errors you encountered and how you fixed them.]
```

---

## 4.5 — Page 1: Executive Overview

**Story this page tells:** "Here is the overall health of the business — revenue trend, scale, and top categories."

**Canvas setup:**
- View → Page View → Fit to Page
- Right-click page tab → Rename → "Executive Overview"
- Format → Canvas background → very light gray (#F5F5F5) or white

**Visuals to build:**

### KPI Cards (top row — 4 cards)
| Card | Measure | Format |
|---|---|---|
| Total Revenue | `[Total Revenue]` | BRL currency, 0 decimals |
| Total Orders | `[Total Orders]` | whole number with comma |
| Avg Order Value | `[Avg Order Value]` | BRL currency, 2 decimals |
| Avg Review Score | `[Avg Review Score]` | 1 decimal, show ★ in title |

**How to build:** Insert → Card → drag measure to Fields well

### Line Chart — Monthly Revenue Trend
- X-axis: `DateTable[MonthName]` (sort by YearMonth)
- Y-axis: `[Total Revenue]`
- Secondary Y-axis: `[Total Orders]` (as a dotted line)
- Title: "Monthly Revenue & Order Volume"
- Add a constant line at the average revenue value

### Bar Chart — Top 10 Categories by Revenue
- Y-axis: `categories[category]`
- X-axis: `categories[revenue]`
- Sort: Descending by revenue
- Title: "Top 10 Product Categories"
- Color: single color (your theme blue)

### Slicer — Year filter
- Field: `DateTable[Year]`
- Style: Dropdown
- Place in top-right corner

**"So what?" text box (add this to every page):**
```
Insert → Text Box → paste this text and edit with your findings:

"Revenue grew [X]% between 2017 and 2018, driven primarily by [top category].
Peak sales occur in [month] — likely linked to promotional events."
```

**Screenshot taken:** `screenshots/page1_executive_overview.png`

**What I observed on this page:**
```
[Write 2-3 observations after building the page.
Example: "The monthly trend confirms my Python finding — clear November 2017 peak.
The top category dominates significantly over #2 and #3, which was less obvious
in the SQL table view. The KPI cards give an immediate sense of business scale."]
```

---

## 4.6 — Page 2: Customer & Delivery Analysis

**Story this page tells:** "Delivery delays are hurting satisfaction — and most customers never come back."

**Setup:**
- Duplicate Page 1 → clear all visuals → rename "Customer & Delivery"

**Visuals to build:**

### Scatter Plot — Delivery Days vs. Review Score
- X-axis: `orders[delivery_days]`
- Y-axis: `orders[review_score]`
- Play Axis or Details: `orders[customer_state]`
- Title: "Delivery Time vs. Customer Satisfaction"
- Add a trend line: Analytics pane → Trend Line → enable

### Clustered Bar — Avg Delivery Days by Review Score
- Y-axis: `orders[review_score]` (as text/categorical)
- X-axis: `[Avg Delivery Days]`
- Conditional formatting: color bars red for scores 1-2, green for 4-5
- Title: "Average Delivery Days per Star Rating"

### Donut Chart — Payment Methods
- Legend: `orders[payment_type]`
- Values: `[Total Orders]`
- Title: "Payment Method Distribution"

### Table — RFM Customer Segments
- Rows: `rfm[Segment]`
- Values: Count of `rfm[customer_unique_id]`, Average of `rfm[Monetary]`, Average of `rfm[Recency]`
- Conditional formatting on Monetary: data bars
- Title: "Customer Segments (RFM Analysis)"

### KPI Cards (top row — 3 cards)
| Card | Measure |
|---|---|
| Avg Delivery Days | `[Avg Delivery Days]` |
| Late Delivery Rate | `[Late Delivery Rate]` — format as % |
| Repeat Customer Rate | `[Repeat Customer Rate]` — format as % |

**"So what?" text box:**
```
"Orders delivered [X]+ days late receive an average of [Y] stars — [Z] stars
lower than on-time orders. Only [repeat rate]% of customers made a repeat purchase,
highlighting a retention challenge."
```

**Screenshot taken:** `screenshots/page2_customer_delivery.png`

**What I observed on this page:**
```
[Write 2-3 observations. Example: "The scatter plot makes the delivery-satisfaction
relationship visually undeniable. The RFM table reveals that [X]% of customers
are in the 'Lost' segment — a finding that surprised me given the platform's
apparent growth in revenue. Revenue grew despite poor retention, meaning the
platform relied almost entirely on new customer acquisition."]
```

---

## 4.7 — Page 3: Seller & Category Performance

**Story this page tells:** "A small number of sellers and categories drive most of the business."

**Setup:**
- Duplicate Page 2 → clear visuals → rename "Seller & Category Performance"

**Visuals to build:**

### 100% Stacked Bar — Category Revenue vs. Rating
- Y-axis: `categories[category]` (top 10 by revenue)
- X-axis: `categories[revenue]`
- Color: conditional on `categories[avg_review]`
- Title: "Category Revenue & Avg Rating"

### Scatter Plot — Revenue vs. Avg Review by Category
- X-axis: `categories[avg_review]`
- Y-axis: `categories[revenue]`
- Size: `categories[units_sold]`
- Details: `categories[category]`
- Title: "Category Quadrant: Revenue vs. Satisfaction"
- Add reference lines at avg_review = 4.0 and revenue = midpoint to create 4 quadrants

### Matrix Table — Category Deep Dive
- Rows: `categories[category]`
- Values: `categories[revenue]`, `categories[units_sold]`, `categories[avg_price]`, `categories[avg_review]`
- Conditional formatting: color scale on all columns
- Title: "Full Category Performance Matrix"

### KPI Cards — top row
| Card | Value |
|---|---|
| Total Sellers | Count of distinct sellers (create a measure) |
| Top Seller Revenue Share | Top seller's revenue / total (create a measure) |

**"So what?" text box:**
```
"The top [X]% of sellers generate 80% of revenue.
Categories in the top-right quadrant (high revenue + high rating) represent
the platform's strongest value propositions — [category] and [category]."
```

**Screenshot taken:** `screenshots/page3_seller_category.png`

**What I observed on this page:**
```
[Write 2-3 observations. The quadrant chart is the most powerful visual on this page.
Which categories fall in each quadrant? What does that tell you about the business?]
```

---

## 4.8 — Apply Consistent Theme

**What I did:**
- View → Themes → Customize current theme

**Color palette used:**
| Element | Color (Hex) | Reason |
|---|---|---|
| Primary color | #1A5276 | Navy blue — professional, matches README |
| Positive/good | #1D9E75 | Teal green |
| Negative/bad | #D85A30 | Coral red |
| Neutral | #888780 | Gray |
| Background | #F8F9FA | Off-white |

**Typography:**
- Title font: Segoe UI, 14px, Bold
- Body/label font: Segoe UI, 11px, Regular
- KPI number font: Segoe UI, 28px, Bold

**Consistency checklist:**
- [ ] All page titles same font and size
- [ ] All KPI cards same size and position (top row)
- [ ] "So what?" text boxes same style on all 3 pages
- [ ] Color usage is consistent (blue = revenue, teal = positive, coral = negative)

---

## 4.9 — Final Review Before Publishing

**Questions to ask yourself about each page:**
```
1. Can someone understand this page in 10 seconds without reading the text box?
2. Is there any visual that doesn't directly support the page's story? (remove it)
3. Are all numbers formatted correctly? (no unformatted decimals, correct currency)
4. Does the page title tell someone what to expect before they read anything else?
```

**Things I changed after final review:**
```
[Write what you adjusted — this shows iterative thinking, which is a sign of a
good analyst. Example: "I removed the geolocation map from Page 2 because the
state-level data for Brazil isn't meaningful in a global context.
I replaced it with the delivery days bar chart which was more impactful."]
```

---

## 4.10 — Summary: What I Built

**Dashboard story in 3 sentences:**
```
[Write this — it's your elevator pitch for the project.
Example: "This dashboard shows that the platform grew revenue consistently
from 2016-2018, but faces two underlying challenges: a [X]% repeat customer
rate suggesting poor retention, and a clear link between late delivery and
1-star reviews. Fixing logistics in the highest-delay regions and re-engaging
at-risk customer segments represent the two highest-impact opportunities."]
```

**Power BI skills demonstrated:**
```
- Data modeling: relationships between 3 tables + date table
- DAX measures: calculated aggregations, DIVIDE, DATEADD, FILTER
- Time intelligence: MoM growth using DATEADD
- Visualization types: KPI cards, line chart, bar chart, scatter, donut, matrix, table
- Design: consistent theme, conditional formatting, data bars, trend lines
- Storytelling: "So what?" text boxes, clear page narrative
```

---

## 4.11 — GitHub Commit

```bash
git add powerbi/ecommerce_dashboard.pbix
git add screenshots/page1_executive_overview.png
git add screenshots/page2_customer_delivery.png
git add screenshots/page3_seller_category.png
git add process/phase4_powerbi.md
git commit -m "Phase 4 complete: Power BI 3-page dashboard — overview, delivery analysis, category performance"
git push origin main
```

**Final step — update README.md:**
- Replace the screenshot placeholder table with your actual screenshot images
- Fill in the Key Findings section with your 3 final insights
- Fill in the Recommendations section with your 3 actionable suggestions

---

*Previous: [Phase 3 — Python](./phase3_python.md)*
*You're done! Go update the [README](../README.md) with your final findings.*
