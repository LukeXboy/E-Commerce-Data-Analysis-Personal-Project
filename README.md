# E-Commerce Revenue & Customer Behavior Analysis

**Independent data analytics portfolio project | October 2026**  
**Tools:** Python, Pandas, SQL, exploratory data analysis, KPI decomposition, customer segmentation  
**Dataset:** [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)  

## Overview

This completed **exploratory and business analytics case study** investigates revenue growth and customer purchasing behavior in a Brazilian e-commerce marketplace. It connects data-quality work, relational data modeling, SQL/Python analysis, and business interpretation to answer:

> **What drove the marketplace's October-to-November 2017 revenue increase, and what did newly observed customers buy?**

**Scope:** Data validation, order-level dataset construction, revenue decomposition, geographic and product-category contributions, new-versus-returning customer classification, category reach, and customer-segment AOV analysis.

**Not in scope:** A Power BI/Tableau dashboard, 90-day repeat-purchase retention, causal campaign attribution, hypothesis testing, or predictive machine learning. These were considered but **were not completed and are not claimed as deliverables**.

The repository includes a **consolidated reproduction notebook** documenting the analytical workflow. Its narrative summarizes earlier observed results; running the notebook with the Olist CSV files is necessary to independently verify those outputs. The notebook is not represented as the original working file.

---

## Executive Summary

| Finding | Result |
|---|---|
| Total raw orders audited | **99,441** distinct orders |
| Delivered-order revenue, October 2017 | **BRL 751,140.27** |
| Delivered-order revenue, November 2017 | **BRL 1,153,528.05** |
| Month-over-month revenue growth | **+53.6%** |
| Unique-customer growth | **+62.6%** |
| Orders per customer | **Approximately unchanged** |
| Average order value (AOV) | **−5.7%** |
| Share of revenue increase from SP, RJ and MG | **Approximately 68%** |
| November 2017 new customers studied in SP/RJ/MG/RS | **5,074** |
| New customers who purchased from any of the five leading categories | **2,025 (39.91%)** |
| AOV: top-five category buyers vs other new buyers | **BRL 143.96 vs BRL 157.66** |

**Bottom line:** November's observed growth was primarily **customer- and order-volume-driven**, not driven by higher AOV or markedly higher orders per customer. Category demand increased broadly, with substantial growth concentrated in large customer markets. These are **descriptive findings**, not causal explanations of the increase.

## Business Questions Answered

1. Was monthly revenue growth driven by more customers, more purchases per customer, or higher order value?
2. Which product categories and states contributed most to the October–November change?
3. Were November buyers in key states mostly new or returning within the observed purchase history?
4. What proportion of new buyers purchased from the five leading product categories?
5. How did order value differ between new buyers of those categories and other new buyers?

---

## Data Sources and Analytical Design

Source tables used in the analytical workflow:

- **Orders:** `order_id`, `customer_id`, order status, timestamps
- **Customers:** `customer_id`, `customer_unique_id`, customer state
- **Payments:** `order_id`, `payment_value`
- **Reviews:** `order_id`, review score
- **Order items:** `order_id`, item identifier, product, item price and freight
- **Products and category translation:** product/category relationships and English labels

### Critical join keys and data grain

| ID / measure | Correct interpretation |
|---|---|
| `order_id` | Identifies one marketplace order |
| `customer_id` | Order-specific customer ID; joins orders to customers |
| `customer_unique_id` | Persistent customer identity; used for unique customers and previous-order history |
| `order_revenue` | Sum of recorded `payment_value` at **one row per order** |
| `price` | **Item-level** product price, used for category/item revenue analysis |

To avoid double counting, payments and reviews were **aggregated by `order_id` before joining** to the orders table. Joining full payment-based order revenue directly onto multiple order-item rows would overstate revenue.

The primary executive KPI population contains **delivered orders only**. Analysis periods use **year-month**, not month number alone. Category analysis uses item price, which is a different definition from order-level payment revenue; the two measures are **not directly interchangeable**.

### Data-quality audit

- **99,441** rows and **99,441 distinct order IDs**; no duplicated order IDs in the base orders table.
- **8** delivered orders lacked customer-delivery timestamps.
- **23** orders had a recorded customer-delivery timestamp before the carrier handoff timestamp.
- After constructing the order-level joins: **99,441 orders retained**, with no duplicate order IDs and no missing `customer_unique_id`.
- One delivered order lacked a recorded payment amount. **BRL 143.46** was reconstructed from three items at BRL 44.99 plus BRL 2.83 freight each. This is explicitly **imputed, not observed payment revenue**.
- **768** orders had no review score after review aggregation; missing review scores were preserved as missing rather than replaced with zero.

Delivery timestamp anomalies were flagged for delivery-related analyses rather than indiscriminately removing otherwise usable orders.

---

## Analysis 1 — October to November 2017 Revenue Decomposition

Using:

```text
Revenue = Unique customers × Orders per customer × Average order value
```

| KPI (delivered orders) | October 2017 | November 2017 | Change |
|---|---:|---:|---:|
| Order revenue (BRL) | 751,140.27 | 1,153,528.05 | **+53.6%** |
| Unique customers | 4,417 | 7,183 | **+62.6%** |
| Orders | 4,478 | 7,289 | **+62.8%** |
| Orders/customer | 1.0138 | 1.0148 | ~flat |
| AOV (BRL) | 167.74 | 158.26 | **−5.7%** |

**Interpretation:** More distinct customers and orders coincided with the revenue increase, while order frequency was essentially flat and AOV fell. Supporting item-level analysis also showed a lower average item price (approximately BRL 122.80 to BRL 116.55) and broadly stable items per order.

Core growth metrics were independently reproduced with **SQL conditional aggregation** as well as Pandas.

## Analysis 2 — Product-Category Contribution

Item revenue by translated product category was compared at a **year-month × category** grain.

| Category | Oct item revenue (BRL) | Nov item revenue (BRL) | Increase (BRL) |
|---|---:|---:|---:|
| `bed_bath_table` | 46,007.70 | 87,957.63 | **41,949.93** |
| `health_beauty` | 40,698.99 | 78,274.40 | **37,575.41** |
| `furniture_decor` | 30,009.74 | 62,091.27 | **32,081.53** |
| `watches_gifts` | 64,874.63 | 95,292.34 | **30,417.71** |
| `toys` | 33,324.42 | 62,611.26 | **29,286.84** |
| `computers_accessories` | 42,009.38 | 69,676.32 | **27,666.94** |

**Interpretation:** The increase was spread across several categories rather than attributable to a single category. This table uses **sum of item prices**, not total order payment value.

## Analysis 3 — Geographic Contribution

The geographic analysis used **order-level payment revenue** associated with each customer's state.

| State | Oct order revenue (BRL) | Nov order revenue (BRL) | Increase (BRL) |
|---|---:|---:|---:|
| SP | 249,924.09 | 401,027.72 | **151,103.63** |
| RJ | 110,577.92 | 172,234.57 | **61,656.65** |
| MG | 93,049.17 | 154,131.98 | **61,082.81** |
| RS | 43,173.06 | 67,262.74 | **24,089.68** |
| SC | 24,773.77 | 44,269.59 | **19,495.82** |
| PR | 37,404.60 | 54,237.96 | **16,833.36** |

**SP, RJ and MG contributed approximately 68%** of the October–November total revenue increase, identifying large regional markets for further analysis. Their contribution does not by itself establish why sales increased.

## Analysis 4 — New vs Returning Buyers

A customer is **new within the observed dataset** if they had **no earlier delivered order** before the relevant month. A returning customer has at least one earlier delivered order in the available history. This definition does not guarantee first purchase ever.

For November 2017:

| State | New customers | Returning customers | Share new |
|---|---:|---:|---:|
| SP | 2,803 | 41 | 98.56% |
| RJ | 979 | 17 | 98.29% |
| MG | 893 | 21 | 97.70% |
| RS | 399 | 6 | 98.52% |
| **Total** | **5,074** | **85** | **98.35%** |

The November four-state order-level subset contained **5,242 distinct delivered orders**, including orders from both groups.

**Interpretation:** New-within-observed-history buyers overwhelmingly dominated the November customer mix in these states. This supports a new-customer inflow explanation without implying that campaign-driven acquisition was proven.

## Analysis 5 — Category Reach Among New Buyers

The five categories with the most distinct new buyers in November 2017 (SP/RJ/MG/RS) were:

| Category | Distinct new buyers | Orders | Items sold | Item revenue (BRL) |
|---|---:|---:|---:|---:|
| `bed_bath_table` | 610 | 621 | 735 | 68,343.26 |
| `furniture_decor` | 388 | 395 | 555 | 43,879.82 |
| `sports_leisure` | 366 | 376 | 412 | 44,080.30 |
| `health_beauty` | 336 | 340 | 373 | 51,087.67 |
| `toys` | 335 | 336 | 375 | 46,704.16 |

A buyer can belong to more than one category. To compute true **combined reach**, the analysis counted **distinct customer IDs across the union** of the five categories instead of adding category-level counts.

| Fixed five-category reach | October 2017 | November 2017 |
|---|---:|---:|
| New customers in SP/RJ/MG/RS | 3,052 | 5,074 |
| Buyers who bought ≥1 item in those categories | 1,176 | 2,025 |
| Share of new buyers reached | **38.53%** | **39.91%** |

- New-customer count: **+66.3%**
- Buyers of the selected categories: **+72.2%**
- Combined category reach: **+1.38 percentage points**

SQL validation used `COUNT(DISTINCT CASE WHEN ... THEN customer_unique_id END)` and a zero-safe denominator.

## Analysis 6 — Customer Segments and AOV

November new buyers in SP/RJ/MG/RS were assigned to mutually exclusive groups:

- **Top-five buyer:** purchased at least one item in the selected five categories, possibly alongside other items.
- **Other buyer:** did not purchase a recorded item from the selected five categories.

Customer labels were deduplicated to **one row per `customer_unique_id`** and then joined to the **order-level** dataset, so complete order payments were not accidentally multiplied by item count.

| KPI | Top-five buyers | Other buyers |
|---|---:|---:|
| Unique new buyers | 2,025 | 3,049 |
| Orders | 2,080 | 3,073 |
| Order revenue (BRL) | 299,439.32 | 484,474.40 |
| AOV (BRL) | **143.96** | **157.66** |

**Observed difference:** Top-five buyers' AOV was approximately **BRL 13.69 (8.69%) lower**.

**Interpretation:** AOV is revenue per order, **not margin or lifetime value**. Lower AOV alone is insufficient justification for reducing marketing investment; customer acquisition costs, profit margins and later purchase behavior would be needed. Untranslated/missing category labels are a potential classification limitation that should be audited before treating every "Other buyer" as a confirmed non-top-five purchaser.

---

## Business Recommendations

These are **investigation priorities**, not established causal findings:

1. **Investigate acquisition and demand drivers:** Break down November growth by traffic, campaigns, promotions and channels *if such data become available*, rather than attributing the growth to marketing based on transaction history alone.
2. **Prioritize market-specific diagnosis:** Start with SP, RJ and MG because of their large observed contributions; examine the operational and marketing contexts before reallocating budget.
3. **Review category strategy using economics, not just reach:** Pair product-category reach and revenue with contribution margin, fulfillment costs and acquisition costs before changing promotional spend.
4. **Evaluate subsequent customer value:** A future 90-day repeat-purchase analysis could help test whether November's new buyers formed sustainable demand, but this analysis **was not performed in the present project**.

## Methods and Quality Controls

- Audited missing values, duplicate keys, order status and timestamp anomalies.
- Defined **order**, **item**, **customer**, and **category-customer** grains before aggregation.
- Used `customer_id` for order/customer joins and `customer_unique_id` for customer-history classification.
- Aggregated one-to-many payment/review records before joining to preserve order-level uniqueness.
- Distinguished **payment-based order revenue** from **item-price category revenue**.
- Checked segment populations and used distinct-count logic for overlapping categories.
- Reproduced selected Pandas metrics in SQL.
- Distinguished observed associations from causes and documented missing-data limitations.

## Reproducibility

The full analysis workflow is available in **[Olist_Ecommerce_Revenue_Customer_Analytics_Final.ipynb](./Olist_Ecommerce_Revenue_Customer_Analytics_Final.ipynb)**. The notebook uses Python, Pandas, NumPy, and an optional SQL cross-check with DuckDB. It was assembled as a reproducible account of earlier analyses; some recorded figures in the narrative should be checked against a fresh run on the source data.

1. Download the [Brazilian Olist E-Commerce dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and extract its CSV files.
2. Clone the repository and install the required libraries (for example, `pip install pandas numpy duckdb jupyter`).
3. **Match the CSV location to the code.** The current first code cell sets `DATA_DIR = Path('.')`, so it looks for CSV files **in the notebook's working directory** (normally the repository root). The notebook's introductory markdown alternatively suggests `data/raw/`; if you place files there, change that code line to `DATA_DIR = Path('data/raw')` before running.
4. Open `Olist_Ecommerce_Revenue_Customer_Analytics_Final.ipynb` and run its cells in order.
5. Reconcile reproduced metrics with the executive summary and data-quality notes in this README. Data files are not committed here.

**Repository contents:** The consolidated notebook and this README are available. No separate dashboard, trained model, or completed 90-day retention analysis is included.

## Project Status

**Completed for the defined exploratory and business analytics scope (October 2026).**

The documented analyses and business interpretation are complete for this case-study scope, and the consolidated notebook is available above. Its recorded analytical results should be verified through a fresh run using the downloaded Olist data. No dashboard, predictive model, formal statistical significance test, or 90-day retention rate is claimed.

---

**Xing Liu (Luke)**  
University of Toronto — Master of Engineering, Data Analytics & Machine Learning
