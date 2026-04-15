# Shield Insurance Analytics Dashboard
### Power BI Pilot Project | AtliQ Technologies Virtual Internship | codebasics

[![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-yellow?logo=powerbi)](https://shorturl.at/b28I0)
[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/nikhil6131/Shield-Insurance-Dashboard)
[![License](https://img.shields.io/badge/License-MIT-blue)](LICENSE)

---

## 📌 Project Overview

Shield Insurance approached AtliQ Technologies with a need to build a centralised analytics solution to monitor business performance across their customer base, revenue streams, and sales channels.

This project delivers a **4-page interactive Power BI dashboard** covering **November 2022 to April 2023**, enabling Shield Insurance's leadership team to make data-driven decisions on customer acquisition, sales strategy, and risk management.

> **Live Dashboard →** [View on Power BI Service](https://shorturl.at/aoeBe)

---

## 🎯 Business Requirements

The client (Mathew, Shield Insurance) requested the following:

| Requirement | Delivered |
|-------------|-----------|
| Track total customers and total revenue | ✅ KPI cards on General Analysis |
| Monitor daily revenue and customer growth rates | ✅ Daily Growth KPI cards |
| Month-over-month % change tracking | ✅ LM CHG % measures on all pages |
| Segment customers by age group | ✅ Age Group Analysis page |
| Analyse revenue and customers by city | ✅ Revenue Split table |
| Revenue and customer trend graphs with toggle | ✅ Trend chart on General Analysis |
| Filters by sales mode, age, city, month, policy ID | ✅ 5 interactive filters |
| Separate Sales Mode Analysis page | ✅ Page 2 |
| Revenue and customer split % by sales mode | ✅ Pie charts + bar charts |
| Sales mode trend over months | ✅ Line charts |
| Separate Age Group Analysis page | ✅ Page 3 |
| Expected settlement analysis by age group | ✅ Settlement Amount table |
| Policy preference by age group | ✅ Policy Preference matrix |
| Sales mode preference by age group | ✅ Age vs Sales Mode matrix |

---

## 📊 Dashboard Pages

### 🏠 Home Page
- Navigation hub with descriptions of each analytical view
- Links to General Analysis, Sales Mode Analysis and Age Group Analysis
- Data period and last upload date

### 📊 General Analysis
- **4 KPI Cards** — Total Revenue, Total Customers, Daily Revenue Growth, Daily Customer Growth
- **LM CHG %** — Last month percentage change for all 4 KPIs
- **Revenue Split** — Total revenue and customers broken down by city
- **Customer Split** — Total customers and revenue broken down by age group
- **Customer Segmentation** — Cross-table: City × Age Group with revenue and customers
- **Customers Trend by Month** — Line chart tracking monthly customer count
- **5 Filters** — Month, Sales Mode, Age, City, Policy ID

### 🛒 Sales Mode Analysis
- **4 KPI Cards** — Total Revenue, Total Customers, LM CHG % Revenue, LM CHG % Customers
- **Customer Split by Sales Mode** — Pie chart showing % split across 4 channels
- **Revenue Split by Sales Mode** — Pie chart showing revenue % split
- **Customer Trends by Month** — Monthly customer count line chart
- **Revenue Trends by Month** — Monthly revenue line chart
- **2 Filters** — Sales Mode, Month

### 👥 Age Group Analysis
- **KPI Cards** — Total Revenue, Settlement Amount
- **Age Group vs Est. Settlement Amount** — Settlement risk table by age group
- **Customers Trend by Age Group** — Multi-line chart (6 age groups over 6 months)
- **Age Group vs Sales Mode** — Customer count matrix
- **Customers by Age Group** — Horizontal bar chart
- **Age Group vs Policy Preferences** — Full matrix (9 policies × 6 age groups)
- **2 Filters** — Month, Policy ID

---

## 🗂️ Data Model

**Star Schema** — fact_premiums as the central fact table connected to 3 dimension tables.

```
dim_customer ──────┐
dim_date ──────────┼──── fact_premiums
dim_policies ──────┘

fact_settlements (standalone — no join required)
```

### Tables

| Table | Rows | Description |
|-------|------|-------------|
| `dim_customer` | 26,841 | Customer details — DOB, city |
| `dim_date` | 181 | Date dimension — daily, monthly, week number |
| `dim_policies` | 9 | Policy types — base coverage and premium |
| `fact_premiums` | 26,841 | Policy transactions — sales mode, premium amount |
| `fact_settlements` | 73 | Expected settlement % by age |

### Calculated Columns (dim_customer)

```dax
Age = INT(DATEDIFF(dim_customer[dob], DATE(2023,4,30), DAY) / 365.25)

Age Group = 
SWITCH(TRUE(),
    dim_customer[Age] < 25, "<25",
    dim_customer[Age] < 35, "25-34",
    dim_customer[Age] < 45, "35-44",
    dim_customer[Age] < 55, "45-54",
    dim_customer[Age] < 65, "55-64",
    "65+")

Age Group Sort = 
SWITCH(dim_customer[Age Group],
    "<25", 1, "25-34", 2, "35-44", 3,
    "45-54", 4, "55-64", 5, "65+", 6, 0)
```

---

## 📐 DAX Measures

All measures stored in a dedicated `_Measures` table.

```dax
-- Core KPIs
Total Revenue = SUM(fact_premiums[final_premium_amt(INR)])
Total Customers = DISTINCTCOUNT(fact_premiums[customer_code])
Total Policies Sold = COUNTROWS(fact_premiums)
Avg Premium = DIVIDE([Total Revenue], [Total Policies Sold], 0)

-- Last Month Comparisons
LM Revenue = CALCULATE([Total Revenue], PREVIOUSMONTH(dim_date[date]))
LM Customers = CALCULATE([Total Customers], PREVIOUSMONTH(dim_date[date]))

-- Month-over-Month %
MoM Revenue % = DIVIDE([Total Revenue] - [LM Revenue], [LM Revenue], 0)
MoM Customer % = DIVIDE([Total Customers] - [LM Customers], [LM Customers], 0)

-- Daily Growth
Daily Rev Growth % = 
VAR today = MAX(dim_date[date])
VAR yesterday = today - 1
VAR todayRev = CALCULATE([Total Revenue], dim_date[date] = today)
VAR yestRev = CALCULATE([Total Revenue], dim_date[date] = yesterday)
RETURN DIVIDE(todayRev - yestRev, yestRev, 0)

Daily Cust Growth % = 
VAR today = MAX(dim_date[date])
VAR yesterday = today - 1
VAR todayCust = CALCULATE([Total Customers], dim_date[date] = today)
VAR yestCust = CALCULATE([Total Customers], dim_date[date] = yesterday)
RETURN DIVIDE(todayCust - yestCust, yestCust, 0)

-- Sales Mode
Revenue % by Mode = 
DIVIDE([Total Revenue],
    CALCULATE([Total Revenue], ALL(fact_premiums[sales_mode])), 0)

Customer % by Mode = 
DIVIDE([Total Customers],
    CALCULATE([Total Customers], ALL(fact_premiums[sales_mode])), 0)

-- Settlement
Avg Settlement % = AVERAGE(fact_settlements[settlement %])
```

---

## 💡 Key Insights

| # | Insight | Impact |
|---|---------|--------|
| 1 | **Delhi NCR** generates ₹402M — 40.6% of total revenue | Highest priority market |
| 2 | **31-40 age group** has 9,872 customers and ₹284M revenue | Core commercial segment |
| 3 | **Offline-Agent** dominates at 55.67% of revenue and customers | Primary acquisition channel |
| 4 | **Online channels** grew from ~3% to ~29% share in 6 months | Key growth opportunity |
| 5 | **March 2023** delivered ₹264M — 85% above the previous month | Seasonal spike to plan for |
| 6 | **65+ group** carries ₹158.53M settlement risk on just 2,399 customers | Highest per-customer risk |
| 7 | **POL4321HEL** is the most popular policy across all age groups | Top product for cross-sell |

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| **Power BI Desktop** | Dashboard development |
| **DAX** | Calculated columns and measures |
| **Power Query** | Data transformation and type management |
| **Power BI Service** | Publishing and sharing |
| **Microsoft Excel** | DAX metrics documentation |

---

## 📁 Repository Structure

```
Shield-Insurance-Dashboard/
│
├── Data/
│   ├── dim_customer.csv
│   ├── dim_date.csv
│   ├── dim_policies.csv
│   ├── fact_premiums.csv
│   └── fact_settlements.csv
│
├── Documentation/
│   ├── Shield_Insurance_DAX_Metrics_List.xlsx
│   ├── Shield_Insurance_Dashboard_Documentation.pptx
│   └── meta_data.txt
│
├── Dashboard/
│   └── Shield_Insurance_Dashboard.pbix
│
├── Presentation/
│   └── Shield_Insurance_Stakeholder_Presentation.pptx
│
└── README.md
```

---

## 🚀 How to Use

1. **Clone the repository**
```bash
git clone https://github.com/nikhil6131/Shield-Insurance-Dashboard.git
```

2. **Open the dashboard**
   - Open `Shield_Insurance_Dashboard.pbix` in Power BI Desktop
   - All data is pre-loaded — no additional setup required

3. **Or view live**
   - [Click here to view the live interactive dashboard](https://shorturl.at/aoeBe)

---

## 📬 Contact

**Nikhil Chhetri**
- LinkedIn: [linkedin.com/in/nikhilchhetrianalytics](https://linkedin.com/in/nikhilchhetrianalytics)
- GitHub: [github.com/nikhil6131](https://github.com/nikhil6131)
- Email: nikhil6131@gmail.com

---

*Built as part of the AtliQ Technologies Virtual Internship — codebasics Data Analyst Bootcamp*

*#codebasicsvirtualinternship*
