# Customer Shopping Behaviour & Trends Analysis

**Satish Mourya** — B.Tech, Artificial Intelligence and Data Science (AIDS), KDK College of Engineering, Nagpur · Class of 2027
Tools: **Python (Pandas) · SQL (PostgreSQL) · Power BI (DAX)**

> Analysed 3,900 retail transactions to understand what drives spending and loyalty — cleaned in Python, answered 17 business questions in SQL, and built an interactive Power BI dashboard.
> **Total Revenue: $233,081 · Avg Purchase Amount: $59.76 · Avg Review Rating: 3.75**

## The question

What makes customers spend more, come back, and subscribe — and where should a retailer push next?

## Dataset

- Source data: `customer_shopping_behavior.csv` — **3,900 rows, 18 columns** (one row = one customer transaction)
- Fields include category, item purchased, purchase amount, age, gender, subscription status, discount applied, promo code used, previous purchases, review rating, shipping type, season
- Cleaned data in this repo: `cleaned_customer_data.csv` (3,900 rows, 19 columns, 0 missing values)

## What I did

### 1. Python (Pandas) — cleaning and feature engineering
Notebook: `Customer_Shopping_Behavior_Analysis.ipynb`
- Found the only missing values: **37 missing review ratings** (3,863 present)
- Filled them with **category-wise median** imputation — ratings differ by category, and the median resists outliers
- Standardised columns to `snake_case` (`purchase_amount_(usd)` → `purchase_amount`)
- Dropped `promo_code_used` — it duplicated `discount_applied` on every row: `(df['discount_applied'] == df['promo_code_used']).all()` is `True`
- Engineered `age_group` with `pd.qcut` (Young Adult / Adult / Middle-aged / Senior) and `purchase_frequency_days` (Weekly = 7, Fortnightly/Bi-Weekly = 14, Monthly = 30, Quarterly/Every 3 Months = 90, Annually = 365)
- Loaded the cleaned DataFrame into **PostgreSQL** (database `customer_behaviour`, table `customer`) using SQLAlchemy + psycopg2

### 2. SQL (PostgreSQL, pgAdmin 4) — 17 business questions
File: `customer_behavior_sql_queries.sql` · Results: `06_SQL_Results.xlsx` (one sheet per question)

| Q | Question |
|---|---|
| Q1 | Revenue by gender |
| Q2 | Discount users who still spent at or above the average (839 customers) |
| Q3 | Top 5 products by average review rating |
| Q4 | Average spend: Standard vs Express shipping |
| Q5 | Subscribers vs non-subscribers — count, average spend, total revenue |
| Q6 | Top 5 products by share of purchases with a discount |
| Q7 | Customer segmentation with a CTE — New (1 previous purchase), Returning (2–10), Loyal (>10) |
| Q8 | Top 3 products per category — `ROW_NUMBER() OVER (PARTITION BY category …)` |
| Q9 | Repeat buyers (>5 previous purchases) — are they subscribed? |
| Q10 | Revenue by age group |
| Q11 | Revenue by category |
| Q12 | Revenue by season |
| Q13 | Revenue by payment method |
| Q14 | Top 5 locations by revenue |
| Q15 | Average spend across all shipping types |
| Q16 | Spend by purchase frequency |
| Q17 | Revenue by size |

Techniques used: CTEs, subqueries, window functions, conditional aggregation.

Extended interview/business question set with queries and verified answers: `Business_Questions_With_Answers.md`

### 3. Power BI — interactive dashboard
File: `customer_behaviour_dashboard.pbix` · Export: `customer_behaviour_dashboard.pdf`
- Connected Power BI to the PostgreSQL database (`customer_behaviour`) in Import mode
- DAX measures created explicitly: Total Customers, Total Revenue, Avg Purchase Amount, Avg Review Rating
- KPI cards: **3,900 customers · $59.76 avg purchase · 3.75 avg rating**
- Visuals: % of customers by subscription status (donut), revenue & sales by category, revenue & sales by age group
- Slicers: subscription status, gender, category, shipping type — all visuals cross-filter

## Key findings (from my own SQL run)

1. **Revenue is volume-driven.** Clothing is the top category with **$104,264 (44.7% of revenue)** from 1,737 orders, while average spend is ~$60 in every category. Top-rated product: Gloves (3.86). Top age group by revenue: Young Adult (**$62,143**).
2. **Subscriptions and discounts do NOT lift spend in this data.** Subscribers average **$59.49** vs $59.87 for non-subscribers. Discounted orders average $59.28 vs $60.13 without a discount. Reporting this honestly matters more than forcing a story.
3. **Loyal customers are the business.** New = 83 customers, Returning = 701, **Loyal = 3,116 customers contributing $185,517 — 79.6% of all revenue.**
4. **The untapped opportunity is countable:** of 3,476 repeat buyers (>5 previous purchases), **2,518 are not subscribed** — a warm audience for a subscription campaign.

## Recommendations

- Protect and reward the Loyal segment first (79.6% of revenue); convert Returning customers (highest average spend, $60.93) into Loyal.
- Target the 2,518 non-subscribed repeat buyers for subscription conversion instead of broad sign-up pushes.
- Tighten discount dependence on Hat (50.0% of orders discounted), Sneakers (49.66%) and Coat (49.07%) — discounts show no basket lift.
- Lead seasonal campaigns with Fall (highest revenue, $60,018) and Clothing; give Outerwear (only 7.95% of revenue) a range/pricing review.

## Project structure

```
README.md                                # this file
Customer_Shopping_Behavior_Analysis.ipynb  # Python cleaning (Pandas)
Customer_Shopping_Behavior_Analysis.html   # same notebook, opens in any browser
CASE_STUDY.md                            # full case study: problem, decisions, findings, recommendations
customer_behavior_sql_queries.sql        # 17 SQL questions (PostgreSQL)
cleaned_customer_data.csv                # cleaned dataset (3,900 x 19)
Business_Questions_With_Answers.md       # extended interview Q&A with verified answers
06_SQL_Results.xlsx                      # output of all 17 queries, one sheet each
customer_behaviour_dashboard.pbix        # Power BI dashboard (source file)
customer_behaviour_dashboard.pdf         # dashboard export — opens anywhere
```

## How to view it (no setup needed)

Open `customer_behaviour_dashboard.pdf` for the dashboard and `06_SQL_Results.xlsx` for every SQL result. To reproduce: run the notebook on the source dataset, load the cleaned data into PostgreSQL as table `customer` in database `customer_behaviour`, then run the SQL file in pgAdmin 4 — every number above will match.

## Credits & links

- Project built following the end-to-end workflow by Amlan Mohanty — [source repo](https://github.com/amlanmohanty1/customer-trends-data-analysis-SQL-Python-PowerBI) · [video](https://www.youtube.com/watch?v=5PrZvPeUw60)
- LinkedIn: https://www.linkedin.com/in/satishkmourya
