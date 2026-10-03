# E-Commerce Data Analysis Personal Project

## Project Overview

This project analyzes the **Brazilian Olist E-Commerce Dataset** to understand how an online marketplace can improve **sustainable revenue growth while maintaining a strong customer experience**.

The project is designed as an end-to-end data analytics portfolio project combining:

- **Python / Pandas** for data cleaning, validation, exploratory analysis, feature engineering, and statistical analysis
- **SQL** for relational analysis, KPI extraction, customer analysis, and business queries
- **Power BI / Tableau** for executive dashboarding and visualization
- **Statistics / Data Science** for deeper investigation of customer behavior and business performance
- **Business analytics** for KPI decomposition, root-cause analysis, and actionable recommendations
- **GitHub documentation** to present the full analytical workflow in a reproducible and recruiter-friendly format

The central business question is:

> **How can an e-commerce marketplace improve sustainable revenue while protecting customer experience?**

---

## Business Objectives

The analysis will focus on questions such as:

1. How are revenue, customers, orders, and average order value changing over time?
2. Is revenue growth primarily driven by customer growth, purchase frequency, or higher order value?
3. Which product categories, sellers, or customer segments contribute most to growth?
4. What factors are associated with lower review scores or poor customer experience?
5. How do delivery performance and order fulfillment relate to customer satisfaction?
6. Which customers are most valuable, and how strong is repeat-purchase behavior?
7. Are there statistically meaningful differences across customer or order groups?
8. What business actions could improve growth without damaging customer experience?

---

## Dataset

This project uses the **Brazilian E-Commerce Public Dataset by Olist**.

The dataset contains multiple relational tables, including:

- Orders
- Customers
- Order items
- Payments
- Reviews
- Products
- Sellers
- Geolocation
- Product category translations

Raw data files are not stored in this repository.

### Key analytical identifiers

- `order_id` — unique order identifier
- `customer_id` — order-level customer identifier
- `customer_unique_id` — persistent customer identifier used for customer-level analysis

A key design decision in this project is to preserve the analytical grain:

> **One row = one order**

One-to-many tables such as payments and reviews are aggregated to the order level before joining.

---

## Project Workflow

```text
Raw relational data
        ↓
Data quality audit
        ↓
Order-level analytical dataset
        ↓
SQL + Python business analysis
        ↓
Customer / revenue / product analysis
        ↓
Statistical analysis
        ↓
Data science modeling
        ↓
Power BI / Tableau dashboard
        ↓
Executive recommendations
```

---

# Current Progress

## 1. Data Quality Audit

The first stage of the project focused on validating whether the order data could be trusted for analysis.

Checks completed include:

- Row and column counts
- Data types
- Missing values
- Duplicate `order_id` values
- Order-status distribution
- Purchase-date range
- Missing delivery timestamps by order status
- Invalid order-lifecycle timestamp sequences
- Join integrity across tables

### Key findings

- **99,441 order records**
- **99,441 unique `order_id` values**
- **0 duplicated order IDs**
- Most missing customer-delivery timestamps are associated with non-delivered orders
- **8 orders** are marked as `delivered` while missing a customer-delivery timestamp
- **23 orders** contain an invalid delivery sequence where the recorded customer-delivery timestamp occurs before the carrier handoff timestamp

These anomalous records are retained for analyses unrelated to delivery duration, but they should be flagged or excluded when calculating logistics-related metrics.

---

## 2. Order-Level Analytical Dataset

The `orders` table is used as the base population.

### Payments

Because an order may contain multiple payment records, payments are first aggregated to one row per order:

```text
order_id
→ SUM(payment_value)
→ order_revenue
```

The aggregated payment table is then left-joined to the orders table.

### Customers

Customer information is joined using:

```text
orders.customer_id = customers.customer_id
```

`customer_unique_id` is used for unique-customer and repeat-purchase analysis.

### Join validation

After joining orders, payments, and customers:

- **99,441 rows**
- **99,441 unique orders**
- **0 duplicated order IDs**
- **0 missing customer_unique_id values**
- **1 missing payment value**

The missing payment belonged to a delivered order with valid item-level records. For this project, its revenue was reconstructed as **BRL 143.46** using item price plus freight. This value is documented as an imputation rather than an observed payment value.

---

## 3. Review Integration

Reviews are aggregated to the order level using:

- Average review score
- Review count

The aggregated review table is left-joined to the order-level analytical dataset.

After the join:

- **99,441 rows preserved**
- **0 duplicated order IDs**
- **768 orders with no review score**

Missing reviews are kept as `NaN` instead of being filled with zero, because a missing review means no review was observed rather than a negative rating.

Missing-review rates are also analyzed by order status to distinguish expected missingness from potential data-quality issues.

---

## 4. Monthly Executive KPI Table

The primary executive KPI population is defined as:

> **Delivered orders only**

This provides a consistent view of fulfilled business activity.

The monthly KPI table contains:

| KPI | Definition |
|---|---|
| Revenue | Sum of order revenue from delivered orders |
| Unique Customers | Distinct `customer_unique_id` |
| Orders | Distinct delivered `order_id` |
| Average Order Value | Revenue / Orders |
| Average Review Score | Mean order-level review score |

Month is defined at the **YYYY-MM** level to preserve chronology across multiple years.

### Initial observations

From the first monthly trend analysis:

- Revenue, unique customers, and order volume generally increase from 2017 onward
- AOV fluctuates within a narrower range than transaction volume
- This suggests that growth is more strongly associated with **volume expansion** than sustained AOV growth
- Very early 2016 months contain extremely small samples and should not be interpreted as representative business periods
- Review scores are generally around 4+, with a weaker period around late 2017 / early 2018 followed by recovery

These are descriptive observations only; further analysis is required before assigning causes.

---

## Next Analysis: Revenue Growth Decomposition

One of the next investigations is the large increase in revenue from **October 2017 to November 2017**.

The first-level decomposition will use:

```text
Revenue
≈
Customers
×
Orders per Customer
×
Average Order Value
```

The analysis will then investigate whether the increase is associated with:

- Customer volume
- Order frequency
- Average order value
- Product/category mix
- Item prices
- Basket size
- Freight
- Seller mix
- Geography

This decomposition is intended to distinguish **what changed** from **why it changed**.

---

# Planned Analysis

The remaining project will expand into the following areas.

## SQL Analysis

Planned SQL work includes:

- Monthly KPI extraction
- Customer-level revenue analysis
- Top customers by market / segment
- Customer retention
- Repeat-purchase behavior
- Product/category performance
- Seller performance
- Window functions and ranking
- Revenue contribution analysis
- Cohort analysis

---

## Python / Pandas Analysis

Planned Python analysis includes:

- Exploratory data analysis
- Revenue-driver decomposition
- Customer behavior analysis
- Product/category mix
- Delivery performance
- Review-score drivers
- Cohort and retention analysis
- Outlier analysis
- Feature engineering

---

## Statistical Analysis

The project will include statistical methods where they answer a meaningful business question, such as:

- Hypothesis testing
- Confidence intervals
- Effect-size interpretation
- Statistical vs practical significance
- Comparing customer/order groups
- Association between delivery performance and review outcomes

The goal is not to apply statistical tests unnecessarily, but to use them to support business decisions.

---

## Data Science Component

A later stage of the project will include a data science component based on the strongest business use case identified during EDA.

Potential directions include:

- Customer segmentation
- Review-score / customer-experience prediction
- Delivery-risk analysis
- Customer-value modeling

The final modeling task will be selected based on business usefulness and data quality rather than simply adding machine learning for its own sake.

---

## Dashboard

A final **Power BI or Tableau executive dashboard** will summarize the most important findings.

Planned dashboard areas include:

- Revenue and order trends
- Customer growth
- Average order value
- Product/category performance
- Geographic performance
- Delivery performance
- Customer review metrics
- Business drill-down filters

---

## Final Deliverables

By the end of the project, this repository is expected to contain:

```text
E-Commerce-Data-Analysis-Personal-Project/
│
├── README.md
├── requirements.txt
│
├── data/
│   └── README.md
│
├── notebooks/
│   ├── 01_data_audit_and_kpi_build.ipynb
│   ├── 02_exploratory_analysis.ipynb
│   ├── 03_statistical_analysis.ipynb
│   └── 04_modeling.ipynb
│
├── sql/
│   ├── data_quality.sql
│   ├── revenue_analysis.sql
│   ├── customer_analysis.sql
│   └── retention_analysis.sql
│
├── dashboard/
│
├── report/
│
└── images/
```

The repository structure will evolve as the project develops.

---

# Technologies

### Languages
- Python
- SQL

### Python Libraries
- Pandas
- NumPy
- Matplotlib
- scikit-learn *(planned for the modeling stage)*

### Analytics & BI
- Power BI / Tableau

### Development
- Jupyter Notebook
- Git
- GitHub

---

## How to Run

1. Clone the repository.

```bash
git clone <YOUR_REPOSITORY_URL>
cd <YOUR_REPOSITORY_FOLDER>
```

2. Install the required Python packages.

```bash
pip install -r requirements.txt
```

3. Download the Brazilian Olist E-Commerce Dataset.

4. Place the raw CSV files in the expected local data directory.

5. Run the notebooks in numerical order.

---

## Analytical Principles Used in This Project

Throughout the project, several principles are followed:

- Define the **grain** before joining tables
- Validate row counts and keys after every important join
- Investigate missing values before imputing or deleting them
- Separate descriptive patterns from causal conclusions
- Check denominators before interpreting averages or rates
- Distinguish statistical significance from practical business significance
- Translate analytical findings into business recommendations
- Document assumptions and data-quality decisions

---

## Project Status

**In progress**

Current milestone:

```text
Data understanding        ✅
Data quality audit        ✅
Order-level dataset       ✅
Review integration        ✅
Monthly KPI table         ✅
Revenue decomposition     🔄
SQL business analysis     ⏳
Customer analysis         ⏳
Statistical analysis      ⏳
Data science modeling     ⏳
Dashboard                 ⏳
Executive report          ⏳
```

---

## Author

**Xing Liu (Luke)**  
Master of Engineering — Data Analytics & Machine Learning  
University of Toronto
