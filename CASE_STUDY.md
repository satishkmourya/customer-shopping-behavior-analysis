# Case Study — Customer Shopping Behaviour & Trends Analysis

**Analyst:** Satish Mourya · **Tools:** Python (Pandas), PostgreSQL (pgAdmin 4), Power BI (DAX)
**Data:** 3,900 retail transactions, 18 raw columns → 19 after cleaning

## 1. Business problem

A retail company sells across four categories (Clothing, Accessories, Footwear, Outerwear) and runs a subscription program plus frequent discounts. Management assumed two things: subscribers spend more, and discounts grow the basket. They asked for evidence on what actually drives revenue, who the valuable customers are, and where the next push should go.

## 2. Approach

1. **Python (Pandas):** loaded the raw data, profiled it (`info`, `describe`, null counts), cleaned it, engineered features, and exported a cleaned dataset.
2. **PostgreSQL:** loaded the cleaned DataFrame into database `customer_behaviour`, table `customer`, via SQLAlchemy, and answered 17 business questions (Q1–Q17) in pgAdmin 4 using GROUP BY, subqueries, CASE-based conditional aggregation, CTEs and window functions.
3. **Power BI:** connected to the PostgreSQL database (Import mode), created explicit DAX measures, and built a one-page dashboard with KPI cards, category/age-group visuals, a subscription donut and cross-filtering slicers.
4. **Verification:** SQL result totals were cross-checked — total revenue $233,081 reconciles across the gender, category, season and age-group breakdowns.

## 3. Data cleaning decisions (and why)

| Issue found | Decision | Reason |
|---|---|---|
| 37 missing review ratings (only nulls in the data) | Fill with the **category-wise median** | Ratings differ by category; median is robust to outliers |
| Mixed-case column names with spaces/brackets | Standardise to **snake_case** | Required for clean SQL |
| `promo_code_used` present alongside `discount_applied` | **Drop `promo_code_used`** | Verified identical on all 3,900 rows — pure duplication |
| Age is a raw number | Create `age_group` with `pd.qcut` (Young Adult / Adult / Middle-aged / Senior) | Segment-level revenue analysis |
| Purchase frequency is text ("Weekly", "Annually"…) | Create `purchase_frequency_days` (7 / 14 / 30 / 90 / 365) | Makes frequency numeric and comparable |

Final cleaned data: **3,900 rows × 19 columns, 0 missing values.**

## 4. Key findings

**F1 — Revenue is volume-driven, and Clothing carries the business.**
Total revenue $233,081. Clothing = $104,264 (44.7%, 1,737 orders), Accessories = $74,200 (31.8%), Footwear = $36,093, Outerwear = $18,524 (only 7.95%). Average purchase is ~$60 in *every* category, so the ranking is decided by order count, not basket size. Single biggest segment: Clothing–Male, $70,628 (30.3% of all revenue).

**F2 — Subscriptions show no spend lift (assumption not supported).**
Subscribers: 1,053 customers, avg spend $59.49, revenue $62,645 (26.9%). Non-subscribers: 2,847 customers, avg $59.87, revenue $170,436. Subscription share of revenue equals its share of customers — the program, as measured here, adds no per-order value.

**F3 — Discounts show no basket lift either.**
Discounted orders (1,677) average $59.28 vs $60.13 for non-discounted orders (2,223). ~43% of orders in every category are discounted. Highest dependence: Hat 50.00%, Sneakers 49.66%, Coat 49.07% of orders discounted.

**F4 — Loyal customers are 4/5th of the business.**
Segmentation by previous purchases — New (1): 83 customers, $4,853 (2.1%). Returning (2–10): 701 customers, $42,711 (18.3%), with the **highest average spend, $60.93**. Loyal (>10): 3,116 customers, **$185,517 = 79.6% of revenue**.

**F5 — A countable growth opportunity.**
3,476 customers are repeat buyers (>5 previous purchases), but only 958 of them (27.6%) are subscribed. **2,518 proven repeat buyers are not subscribed** — the warmest possible audience for subscription conversion.

**F6 — Supporting patterns.**
Top age group: Young Adult, $62,143 (also highest avg spend, $60.45). Best season: Fall, $60,018 (avg $61.56); weakest: Summer, $55,777. Top-rated products: Gloves 3.86, Sandals 3.84, Boots 3.82 (lowest: Shirt 3.62). Men contribute more revenue ($157,890) only because there are 2,652 of them vs 1,248 women — per order, women spend slightly more ($60.25 vs $59.54). Size M alone contributes $105,167. Premium shipping users spend marginally more (2-Day $60.73, Express $60.48 vs Standard $58.46).

## 5. Recommendations

1. **Retention before acquisition:** protect the Loyal segment (79.6% of revenue) with loyalty benefits; specifically convert Returning customers — they already spend the most per order.
2. **Subscription campaign with a precise target:** the 2,518 non-subscribed repeat buyers, not a broad sign-up push. Pair it with a review of subscriber benefits, since current subscribers show no spend advantage (F2).
3. **Tighten discounts** on Hat, Sneakers and Coat first; with no basket lift (F3), blanket discounting is margin leakage.
4. **Seasonal planning:** lead with Fall + Clothing; build a Summer support plan. Review Outerwear range/pricing (weakest on both volume and revenue).
5. **Merchandising:** promote top-rated products (Gloves, Sandals, Boots) in recommendations; investigate Shirt (lowest rated at scale).

## 6. Limitations (stated honestly)

- One transaction snapshot: no dates, so no true time-series or cohort analysis is possible; "loyalty" is proxied by previous-purchase counts.
- Spend averages are very flat across most groups ($59–$61); several differences are small and should not be over-claimed.
- No cost/margin data: revenue findings are not profit findings, including the discount analysis.

## 7. Files in this repository

| File | Contents |
|---|---|
| `Customer_Shopping_Behavior_Analysis.ipynb` / `.html` | Python cleaning notebook (code + outputs; HTML opens in any browser) |
| `customer_behavior_sql_queries.sql` | All 17 SQL questions (PostgreSQL) |
| `cleaned_customer_data.csv` | Cleaned dataset, 3,900 × 19 |
| `06_SQL_Results.xlsx` | Output of every query, one sheet per question |
| `customer_behaviour_dashboard.pbix` / `.pdf` | Power BI dashboard and its PDF export |
| `Business_Questions_With_Answers.md` | Extended interview-level question set with verified answers |
| `CASE_STUDY.md` | This document |
