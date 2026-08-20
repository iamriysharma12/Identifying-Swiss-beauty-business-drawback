# Swiss Beauty India — End-to-End Analytics Project Blueprint
### (Portfolio Project for SQL + Power BI + Excel — built to get you hired)

---

## 0. Why this is a strong portfolio project

Most candidates submit "sales dashboard" projects. Yours has three things that make a PM/hiring manager stop scrolling:

1. **A real risk finding** — 99.4% single-supplier dependency (China, Global Beauty Group) is a genuine business vulnerability, not a vanity metric.
2. **A real whitespace finding** — 40% revenue from North India, only 20% each from West/East/South → clear expansion opportunity, provable with data.
3. **A real product-market-fit gap** — 95% makeup / 5% skincare, and (per your note) shade ranges likely skewed toward lighter tones while ~70% of India's population has medium-to-deep skin tones (Fitzpatrick IV–VI / Indian tone families: wheatish, dusky, deep brown). This is a quantifiable, fixable gap.

Frame the whole project around **one central question**:
> "Should Swiss Beauty diversify supply, expand geographically, and rebalance its product/shade portfolio — and if so, where first, and what's the expected impact?"

Everything below (KPIs, model, dashboards) should answer pieces of that question.

---

## 1. Project Title Options (pick one for your resume/GitHub)

- "Geographic & Product Whitespace Analysis for a D2C Beauty Brand (SQL + Power BI)"
- "Supply Chain Risk & Market Expansion Analytics — Swiss Beauty India Case Study"
- "Shade Inclusivity Gap Analysis: Using Data to Expand a Beauty Brand's TAM"
- "From 40% to Pan-India: A Data-Driven Expansion Strategy for a Beauty D2C Brand"

One-liner for your resume:
> "Built an end-to-end analytics pipeline (SQL → Power BI/Excel) analyzing supplier concentration, geographic revenue skew, and shade/product-mix gaps for a beauty brand; identified prioritized expansion markets and a shade-range strategy projected to increase addressable market by X%."

---

## 2. Business Problem Statement (put this on slide/page 1)

Swiss Beauty (₹383 Cr revenue, FY25) is:
- **Supply-concentrated**: 99.4% of finished goods sourced from a single Chinese supplier (Global Beauty Group) — a single point of failure.
- **Geography-concentrated**: 40% revenue from North India (Delhi, Rajasthan, UP-heavy), only 20% each from West, East, South. 70% from Tier 1/2 cities.
- **Channel-concentrated**: 65% offline general trade (25,000+ stores), implying ~35% online/modern trade/D2C — underleveraged given India's beauty e-commerce growth.
- **Product-concentrated**: 95% makeup, 5% skincare — skincare is structurally underweight vs. category trends.
- **Competing on price** against unrecognized/local brands, not on brand equity — meaning shade range, formulation, and availability are the real levers, not just marketing spend.

**Objective:** Build a data model and set of dashboards that quantify these concentrations, identify the highest-ROI expansion regions, and recommend a shade/product roadmap — then validate with variance, cohort, and outlier analysis.

---

## 3. Data Model (Star Schema)

Since you likely don't have raw transactional data, you'll build a **synthetic but realistic dataset** that matches these Pareto ratios (I can generate this for you — see Section 12). Structure it as a proper star schema; this is what interviewers actually check for.

### Fact Tables

| Fact Table | Grain (1 row = ) | Key Measures |
|---|---|---|
| `Fact_Sales` | one SKU sold at one store/online-order on one date | qty, gross_revenue, discount, net_revenue, cost, margin |
| `Fact_Inventory` | SKU × store/warehouse × date (daily snapshot) | opening_stock, closing_stock, stockout_flag |
| `Fact_Returns` | one return line | qty_returned, reason_code, refund_amount |
| `Fact_Marketing_Spend` | campaign × channel × month | spend, impressions, clicks, conversions |
| `Fact_Online_Events` | one web/app event | session_id, event_type, product_viewed, add_to_cart, purchase_flag |
| `Fact_Supplier_Shipment` | supplier × PO × date | units_shipped, lead_time_days, defect_rate, cost |

### Dimension Tables

| Dim Table | Key Attributes |
|---|---|
| `Dim_Product` | product_id, category (makeup/skincare), sub_category, shade_name, shade_family (fair/wheatish/dusky/deep), price_band, launch_date |
| `Dim_Store` | store_id, store_name, channel (GT/MT/online), city, state |
| `Dim_Geography` | city, state, zone (North/South/East/West/Central), tier (1/2/3) |
| `Dim_Date` | date, week, month, quarter, FY, is_festival_season (Diwali/wedding season flags — huge for beauty) |
| `Dim_Customer` (for online) | customer_id, first_purchase_date, acquisition_channel, city |
| `Dim_Supplier` | supplier_id, supplier_name, country, category_supplied, contract_start |
| `Dim_Channel` | channel_id, channel_type (GT/MT/E-comm/D2C site), platform_name |

### Relationships / Joins
- `Fact_Sales.product_id → Dim_Product.product_id`
- `Fact_Sales.store_id → Dim_Store.store_id`
- `Dim_Store.city → Dim_Geography.city` (snowflake off Dim_Store, or flatten into Dim_Store — flatten for Power BI simplicity)
- `Fact_Sales.date → Dim_Date.date`
- `Fact_Supplier_Shipment.supplier_id → Dim_Supplier.supplier_id` and `.product_id → Dim_Product.product_id`
- `Fact_Online_Events.customer_id → Dim_Customer.customer_id`

All facts join to `Dim_Date`, `Dim_Product`, `Dim_Geography` — this lets one Power BI model answer sales, supply, and marketing questions from shared filters (a "conformed dimension" setup — mention this term explicitly in your interview, it signals real modeling knowledge).

---

## 4. KPI / Metrics Framework

Group these into four scorecards — this maps directly to four Power BI dashboard pages.

### A. Supply Risk KPIs
- **Supplier Concentration Ratio (CR1)** = revenue-weighted % of goods from top 1 supplier (your 99.4%)
- **Herfindahl-Hirschman Index (HHI)** on supplier base — even with few suppliers, shows how concentrated
- **Average Lead Time** and **Lead Time Variance** by supplier
- **Defect/Return Rate** by supplier

### B. Geographic KPIs
- **Revenue Contribution %** by zone (North/South/East/West/Central) and by tier (1/2/3)
- **Revenue per Store** by zone (efficiency, not just size)
- **Penetration Index** = (Zone's % of India population or beauty-market spend) vs (Zone's % of Swiss Beauty revenue) — a ratio >1 means underpenetrated relative to opportunity. **This is your single most important derived metric** — it's what tells you where to expand first.
- **Store Density** (stores per 100k population) by zone

### C. Product / Shade KPIs
- **Category Mix %** (makeup vs skincare)
- **Shade Coverage Index (custom KPI you build)**: number of active SKUs per shade family (fair/medium/dusky/deep) ÷ estimated population share of that skin-tone family in India. A low index for "dusky/deep" = quantified proof of the gap.
- **SKU Productivity** = revenue per active SKU, by shade family — shows if the few deep-shade SKUs that exist are actually selling well (validates demand before recommending expansion)
- **Sell-through Rate** = units sold ÷ units shipped, by shade family

### D. Channel / Online KPIs
- **Channel Mix %** (GT / MT / Online)
- **Online Conversion Rate**, **Cart Abandonment Rate**, **CAC**, **AOV**
- **Repeat Purchase Rate** and **Customer Retention by Cohort** (Section 7)
- **Revenue per Marketing ₹ spent (ROAS)** by channel

---

## 5. Granularity Decisions (state these explicitly in your project doc — interviewers love seeing this reasoning)

| Layer | Grain |
|---|---|
| Sales fact | Line-item (SKU-store-date) — never pre-aggregate, always roll up in DAX/SQL |
| Inventory fact | Daily snapshot per SKU-location |
| Marketing fact | Monthly per campaign-channel (weekly if you want more resolution) |
| Online events | Per-event (session-level) — needed for funnel/cohort analysis |

State clearly: *"All dashboards aggregate up from atomic grain; no pre-aggregated tables were used as source, to preserve drill-down capability."*

---

## 6. Segmentation

1. **Geographic segmentation**: Zone × Tier (5 zones × 3 tiers = 15 segments) — this is your expansion prioritization grid.
2. **RFM Customer Segmentation** (for online/D2C data): Recency, Frequency, Monetary → Champions / Loyal / At-Risk / Lost. Standard SQL window-function exercise, very commonly asked about in interviews.
3. **Product portfolio segmentation (BCG-style, adapted)**: plot each SKU/category on *Growth Rate* (YoY) vs *Revenue Share* → Stars (grow), Cash Cows (defend), Question Marks (test — this is where deep-shade shades sit today), Dogs (rationalize).
4. **Store segmentation**: by revenue per store and growth trend → Top performer / Stable / Underperforming / New.

---

## 7. Variance Analysis

- **YoY Variance**: FY24 vs FY25 revenue, by zone, by category — flag which zones grew fastest as a leading indicator of untapped demand (a small zone growing 40% YoY signals opportunity even if its absolute % is low).
- **Budget/Forecast vs Actual**: if you set a synthetic target (e.g., "20% revenue from South by FY26"), track actual vs. plan monthly.
- **Price Variance**: average selling price vs. list price by channel — general trade often has margin leakage from discounting; quantify it.
- **Mix Variance vs Volume Variance** (classic FP&A decomposition): was revenue change driven by selling more units, or by selling a richer/leaner mix? Use the standard formula:
  `Total Variance = Volume Variance + Mix Variance + Price Variance`

---

## 8. Cohort Analysis

- **Acquisition cohorts** (online customers grouped by first-purchase month) → track **Month 1, 2, 3... retention %** in a cohort triangle/heatmap. This is a classic, high-signal SQL + Power BI exercise (window functions: `DATEDIFF`, `FIRST_VALUE`, self-joins).
- **Cohort by acquisition channel** (paid social vs organic vs offline-to-online) — shows which channel brings the highest-LTV customer, informing where to put incremental marketing ₹ as you expand into new zones.
- **Cohort by first-purchase category** (makeup-first vs skincare-first buyers) — tests whether skincare buyers have higher repeat rate (common in the category), which would justify the skincare expansion strategically, not just anecdotally.

---

## 9. Outlier Detection

- **Store-level outliers**: stores with revenue >2 std dev from their tier's mean — investigate (fraud? stockout misreport? genuine local demand signal worth replicating?).
- **SKU outliers**: shades with abnormally high sell-through despite low distribution — a strong, data-backed signal for which shade tones to scale first (ties directly back to your shade strategy).
- **Return-rate outliers**: SKUs/batches with defect rates >2x category average — possible supplier quality issue, reinforcing the supplier-risk narrative.
- Methods to actually use: **Z-score** and **IQR (1.5×IQR rule)** — implement both in SQL (window functions: `AVG() OVER()`, `STDDEV() OVER()`) and validate visually in Power BI with a scatter/box plot.

---

## 10. SQL Project Plan (what to actually build — this is your "hard skills" evidence)

Build these as a portfolio of ~10-15 queries, saved in a GitHub repo with comments. Examples:

**Supplier concentration (HHI):**
```sql
SELECT 
  SUM(POWER(supplier_revenue_share, 2)) AS HHI
FROM (
  SELECT supplier_id,
         SUM(net_revenue) * 1.0 / SUM(SUM(net_revenue)) OVER () AS supplier_revenue_share
  FROM Fact_Supplier_Shipment
  GROUP BY supplier_id
) t;
```

**Geographic Pareto (ABC analysis):**
```sql
WITH zone_rev AS (
  SELECT g.zone, SUM(f.net_revenue) AS revenue
  FROM Fact_Sales f
  JOIN Dim_Store s ON f.store_id = s.store_id
  JOIN Dim_Geography g ON s.city = g.city
  GROUP BY g.zone
),
ranked AS (
  SELECT *, 
    SUM(revenue) OVER (ORDER BY revenue DESC) * 1.0 / SUM(revenue) OVER () AS cum_pct
  FROM zone_rev
)
SELECT * FROM ranked;
```

**Shade Coverage Index:**
```sql
SELECT p.shade_family,
       COUNT(DISTINCT p.product_id) AS active_skus,
       SUM(f.net_revenue) AS revenue,
       SUM(f.net_revenue) / SUM(f.units_sold) AS revenue_per_unit
FROM Fact_Sales f
JOIN Dim_Product p ON f.product_id = p.product_id
GROUP BY p.shade_family
ORDER BY active_skus ASC;
```

**RFM segmentation:**
```sql
WITH rfm AS (
  SELECT customer_id,
         DATEDIFF(day, MAX(order_date), GETDATE()) AS recency,
         COUNT(DISTINCT order_id) AS frequency,
         SUM(net_revenue) AS monetary
  FROM Fact_Sales
  WHERE channel_type = 'Online'
  GROUP BY customer_id
)
SELECT *,
  NTILE(5) OVER (ORDER BY recency DESC) AS r_score,
  NTILE(5) OVER (ORDER BY frequency) AS f_score,
  NTILE(5) OVER (ORDER BY monetary) AS m_score
FROM rfm;
```

**Cohort retention:**
```sql
WITH first_purchase AS (
  SELECT customer_id, MIN(order_date) AS cohort_month
  FROM Fact_Sales GROUP BY customer_id
),
activity AS (
  SELECT f.customer_id, fp.cohort_month,
         DATEDIFF(month, fp.cohort_month, f.order_date) AS month_number
  FROM Fact_Sales f
  JOIN first_purchase fp ON f.customer_id = fp.customer_id
)
SELECT cohort_month, month_number, COUNT(DISTINCT customer_id) AS active_customers
FROM activity
GROUP BY cohort_month, month_number
ORDER BY cohort_month, month_number;
```

**Outlier detection (Z-score):**
```sql
WITH store_stats AS (
  SELECT store_id, SUM(net_revenue) AS store_revenue,
         AVG(SUM(net_revenue)) OVER () AS avg_rev,
         STDEV(SUM(net_revenue)) OVER () AS std_rev
  FROM Fact_Sales GROUP BY store_id
)
SELECT *, (store_revenue - avg_rev) / std_rev AS z_score
FROM store_stats
WHERE ABS((store_revenue - avg_rev) / std_rev) > 2;
```

---

## 11. Power BI Dashboard Structure (4-5 pages)

1. **Executive Overview** — revenue, YoY growth, category mix, zone mix, supplier concentration callout (big single-number card: "99.4% from 1 supplier")
2. **Geographic Whitespace** — India map (zone-colored by Penetration Index), zone × tier matrix, store density, YoY growth by zone (bubble chart: size=revenue, x=growth%, y=penetration index — this single chart tells the whole expansion story)
3. **Product & Shade Strategy** — Shade Coverage Index chart, SKU productivity by shade family, sell-through outliers, category mix trend
4. **Channel & Online Performance** — funnel (view→cart→purchase), cohort retention heatmap, ROAS by channel, RFM segment sizes
5. **Supplier Risk** — HHI trend, lead time variance, defect rate by supplier, scenario toggle ("what if top supplier lead time +30 days")

Use bookmarks/what-if parameters for a "scenario slider" — e.g., drag a parameter to simulate "% revenue shift to South zone" and watch total revenue recompute. This is a strong interactive touch that PMs specifically respond well to.

---

## 12. Excel Companion Workbook

- Pivot tables mirroring the Pareto cuts (supplier, geography, channel, product) as a "raw analysis" layer before BI polish
- A **What-If Data Table** modeling: if South/East/West each grow to 25% revenue share, what's total revenue uplift?
- A **Sensitivity table** on supplier risk: cost impact if a 2nd supplier is onboarded at a 10-15% cost premium but reduces HHI by X — a real trade-off table a PM would actually use.

---

## 13. Strategic Recommendations (write these as your final "insights" page — tie every point back to a specific chart/metric above)

1. **Geographic**: Prioritize expansion using the Penetration Index — likely South and West first (higher urban beauty spend per capita, growing e-commerce penetration) before East/Central, sequenced by store-economics not just population.
2. **Shade range**: Launch 8-12 new foundation/concealer/compact shades targeting "wheatish-deep" and "deep" undertones (warm and neutral undertones common in South and East India), validated first via the Shade Coverage Index + SKU productivity outliers (i.e., prove the existing few deep shades already overperform before recommending scale-up).
3. **Product mix**: Grow skincare from 5%→15% over 2-3 years, prioritizing SKUs proven to have higher repeat/cohort retention.
4. **Channel**: Shift incremental marketing spend toward online/D2C in Tier 2/3 cities in South/West — cheaper CAC than opening new general-trade relationships in unfamiliar territory.
5. **Supply chain**: Onboard a second supplier (even at 10-20% of volume) specifically for the new shade SKUs — reduces HHI, and de-risks the newest, least-proven part of the portfolio first.

---

## 14. Since you don't have real data — generate a synthetic dataset

I can build you a realistic synthetic dataset next (Python script producing CSVs matching all these Pareto ratios — 99.4% supplier, 40/20/20/20 zones, 65/35 channel, 95/5 category — with enough noise/seasonality/outliers baked in to make the analysis genuinely interesting) plus the actual SQL schema (CREATE TABLE scripts) so you can load it into SQLite/Postgres and start querying today.

Want me to generate that dataset + schema next?
