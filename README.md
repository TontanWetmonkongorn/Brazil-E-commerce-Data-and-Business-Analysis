# 🛒 E-Commerce Sales Analytics
### End-to-End Data Analytics Project | SQL · Python · Excel · Power BI

![Project Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![Tools](https://img.shields.io/badge/Tools-SQL%20%7C%20Python%20%7C%20Power%20BI%20%7C%20Excel-blue)
![Dataset](https://img.shields.io/badge/Dataset-Olist%20E--Commerce%20(Kaggle)-orange)

---

## 📌 Project Overview

This project explores e-commerce sales performance through a full analytics pipeline — from raw data cleaning to an interactive Power BI dashboard. Using the **Olist Brazilian E-Commerce Public Dataset** (Kaggle) as a proxy for emerging market e-commerce behavior, the analysis surfaces actionable insights relevant to online retail growth strategies.

**Central business question:**
> *"What drives e-commerce sales performance, and what should a seller do differently to grow revenue?"*

---

## 🎯 Objectives

- Identify revenue trends, peak sales periods, and top-performing categories
- Understand the relationship between delivery time and customer satisfaction
- Segment customers using RFM analysis to identify high-value and at-risk groups
- Determine which sellers drive the most revenue and why
- Present findings in a clear, decision-ready Power BI dashboard

---

## 🗂️ Repository Structure

```
ecommerce-sales-analytics/
│
├── README.md
│
├── data/
│   └── data_dictionary.xlsx          # Table descriptions and column definitions
│
├── excel/
│   └── data_issues_log.xlsx          # Documented data quality issues before cleaning
│
├── sql/
│   ├── 01_monthly_revenue.sql        # Revenue trend by month
│   ├── 02_top_categories.sql         # Top product categories by revenue
│   ├── 03_repeat_rate.sql            # Customer repeat purchase rate
│   └── 04_delivery_vs_rating.sql     # Delivery time vs. review score
│
├── python/
│   └── eda_analysis.ipynb            # Full exploratory data analysis notebook
│
├── powerbi/
│   └── ecommerce_dashboard.pbix      # Interactive Power BI dashboard
│
└── screenshots/
    ├── page1_executive_overview.png
    ├── page2_customer_delivery.png
    └── page3_seller_performance.png
```

---

## 🛠️ Tools & Why Each Was Used

| Tool | Role in This Project |
|------|----------------------|
| **Excel** | Initial data inspection, data issues log, and sanity-check pivot tables before any code |
| **SQL (SQLite)** | Structured business queries — revenue trends, category rankings, repeat rates, delivery analysis |
| **Python (pandas, seaborn, matplotlib)** | Deep EDA — seasonality analysis, RFM segmentation, correlation analysis, Pareto distribution |
| **Power BI** | Interactive 3-page dashboard with DAX measures, slicers, and storytelling layout |

---

## 📊 Dashboard Preview

> *Screenshots will be added upon project completion*

| Page 1 — Executive Overview | Page 2 — Customer & Delivery | Page 3 — Seller Performance |
|---|---|---|
| *(screenshot)* | *(screenshot)* | *(screenshot)* |

---

## 🔍 Key Findings

> *To be updated as analysis is completed*

1. **Seasonality:** Sales peak in [Month] driven by [Category] — suggesting targeted promotions during this window could lift revenue by X%.
2. **Delivery & Satisfaction:** Orders delivered beyond [X] days receive significantly lower review scores (avg [Y] vs [Z] for on-time orders), particularly in [Region].
3. **Seller Concentration:** The top 20% of sellers generate approximately 80% of total revenue, with key differentiators being [factor 1] and [factor 2].

---

## 💡 Recommendations

> *To be updated as analysis is completed*

1. **Prioritize fast-shipping categories** during peak months — late delivery is the single strongest predictor of low review scores.
2. **Re-engage at-risk customers** (RFM segment) with targeted offers — they represent [X]% of the customer base but have not purchased in [Y] days.
3. **Support mid-tier sellers** with better logistics tools — a small improvement in their delivery rate could meaningfully shift overall platform performance.

---

## 📁 Dataset

- **Source:** [Olist Brazilian E-Commerce Public Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — Kaggle
- **Size:** ~100,000 orders, 9 relational tables
- **Period:** September 2016 – October 2018
- **Note:** This dataset originates from the Brazilian market and is used here to practice end-to-end analytics techniques applicable to e-commerce platforms globally, including emerging markets in Southeast Asia.

---

## ▶️ How to Run This Project

**SQL**
```bash
# Load CSVs into SQLite
sqlite3 olist.db
.mode csv
.import data/olist_orders.csv orders
# Then run any .sql file in /sql folder
```

**Python**
```bash
pip install pandas matplotlib seaborn numpy jupyter
jupyter notebook python/eda_analysis.ipynb
```

**Power BI**
- Open `powerbi/ecommerce_dashboard.pbix` in Power BI Desktop (free)
- Update the data source path to your local `/data` folder if needed

---

## 👤 About

**Wiwat Wetmongkongorn**
Business and Supply Chain Analytics — SIIT, Thammasat University (CGPA 3.48)

[![Email](https://img.shields.io/badge/Email-6622771630%40g.siit.tu.ac.th-red)](mailto:6622771630@g.siit.tu.ac.th)

---

*This project is part of a personal analytics portfolio built to demonstrate end-to-end data skills for internship applications.*

