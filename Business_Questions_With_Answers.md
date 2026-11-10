# Real Business / Interview SQL Questions — With Query + Verified Answer
Dataset: Satish's own cleaned_customer_data.csv — 3,900 rows, 19 columns, 0 nulls (verified)
Table in pgAdmin: customer | Database: customer_behavior | PostgreSQL
Key totals to remember: Total Revenue = $233,081 | Customers = 3,900 | Avg Purchase = $59.76 | Avg Rating = 3.75

NOTE — THE HONEST STORY OF THIS DATA (say this in interviews, it impresses):
Most group averages sit between $59–$61. So volume (how many customers) drives revenue here, NOT higher spend per order. Subscriptions and discounts do NOT lift average spend in this data. A good analyst reports that, instead of inventing a story.

=====================================================================

## B1. "Which category actually makes our money, and is it because of price or volume?"
Business meaning: Category strategy / where to focus range and marketing.
```sql
SELECT category,
       COUNT(customer_id) AS orders,
       ROUND(AVG(purchase_amount),2) AS avg_purchase,
       SUM(purchase_amount) AS total_revenue,
       ROUND(100.0 * SUM(purchase_amount) / SUM(SUM(purchase_amount)) OVER(), 2) AS revenue_share_pct
FROM customer
GROUP BY category
ORDER BY total_revenue DESC;
```
ANSWER:
| Category | Orders | Avg $ | Revenue $ | Share % |
|---|---|---|---|---|
| Clothing | 1,737 | 60.03 | 104,264 | 44.73 |
| Accessories | 1,240 | 59.84 | 74,200 | 31.83 |
| Footwear | 599 | 60.26 | 36,093 | 15.49 |
| Outerwear | 324 | 57.17 | 18,524 | 7.95 |
Takeaway: Clothing + Accessories = 76.6% of revenue, but all avg spends are ~$60. Clothing wins on VOLUME (1,737 orders), not price. Outerwear is weak on both.

## B2. "Male customers bring more revenue — should we target only men?"
```sql
SELECT gender, COUNT(customer_id) AS customers,
       ROUND(AVG(purchase_amount),2) AS avg_purchase,
       SUM(purchase_amount) AS total_revenue
FROM customer GROUP BY gender ORDER BY total_revenue DESC;
```
ANSWER: Male — 2,652 customers, avg $59.54, revenue $157,890. Female — 1,248 customers, avg $60.25, revenue $75,191.
Takeaway: Men bring more ONLY because there are 2x more of them. Per order, women spend slightly MORE ($60.25 vs $59.54). Cutting female targeting would be a wrong decision.

## B3. "What is our single biggest customer segment by revenue?" (Category × Gender)
```sql
SELECT category, gender, COUNT(customer_id) AS customers,
       SUM(purchase_amount) AS revenue
FROM customer
GROUP BY category, gender
ORDER BY revenue DESC LIMIT 3;
```
ANSWER: 1) Clothing–Male $70,628 (1,181 customers)  2) Accessories–Male $50,381 (848)  3) Clothing–Female $33,636 (556)
Takeaway: Clothing–Male alone is 30.3% of total company revenue ($70,628 / $233,081).

## B4. "Does our subscription program actually make customers spend more?" ⭐ INTERVIEW FAVOURITE
```sql
SELECT subscription_status, COUNT(customer_id) AS customers,
       ROUND(AVG(purchase_amount),2) AS avg_spend,
       SUM(purchase_amount) AS total_revenue,
       ROUND(100.0 * SUM(purchase_amount) / SUM(SUM(purchase_amount)) OVER(), 2) AS revenue_share_pct
FROM customer GROUP BY subscription_status;
```
ANSWER: Yes (subscribed) — 1,053 customers (27.0%), avg $59.49, revenue $62,645 (26.88%). No — 2,847 customers (73.0%), avg $59.87, revenue $170,436 (73.12%).
Takeaway: NO — subscribers spend $0.38 LESS per order on average. Subscription share of revenue (26.88%) is exactly its share of customers (27%). In this data the program adds no spend lift — recommend checking subscriber benefits/engagement before pushing more sign-ups. This "data contradicts the assumption" answer is a strong interview answer.

## B5. "Are discounts growing the basket, or are we just giving margin away?"
```sql
SELECT discount_applied, COUNT(customer_id) AS orders,
       ROUND(AVG(purchase_amount),2) AS avg_spend,
       SUM(purchase_amount) AS total_revenue
FROM customer GROUP BY discount_applied;
-- plus discount dependence by category:
SELECT category,
       ROUND(100.0 * SUM(CASE WHEN discount_applied='Yes' THEN 1 ELSE 0 END)/COUNT(*),2) AS discount_pct
FROM customer GROUP BY category ORDER BY discount_pct DESC;
```
ANSWER: Discount Yes — 1,677 orders, avg $59.28. Discount No — 2,223 orders, avg $60.13. Discount usage by category: Outerwear 44.44%, Accessories 43.79%, Footwear 43.24%, Clothing 42.08%. Highest discount-dependent products: Hat 50.00%, Sneakers 49.66%, Coat 49.07%.
Takeaway: Discounted orders spend $0.85 LESS, not more. ~43% of all orders use a discount with no basket lift — discount strategy needs tightening, especially Hat / Sneakers / Coat.

## B6. "How dependent are we on loyal customers?" (Segmentation — CTE)
```sql
WITH seg AS (
  SELECT *,
    CASE WHEN previous_purchases = 1 THEN 'New'
         WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
         ELSE 'Loyal' END AS customer_segment
  FROM customer)
SELECT customer_segment, COUNT(*) AS customers,
       ROUND(AVG(purchase_amount),2) AS avg_spend,
       SUM(purchase_amount) AS revenue,
       ROUND(100.0 * SUM(purchase_amount) / SUM(SUM(purchase_amount)) OVER(), 2) AS revenue_share_pct
FROM seg GROUP BY customer_segment ORDER BY revenue DESC;
```
ANSWER: Loyal — 3,116 customers (79.90%), avg $59.54, revenue $185,517 (79.59%). Returning — 701 (17.97%), avg $60.93 (highest avg), revenue $42,711 (18.32%). New — 83 (2.13%), avg $58.47, revenue $4,853 (2.08%).
Takeaway: 4 out of every 5 dollars come from Loyal customers — retention is the business. Returning customers spend the most per order, so converting Returning → Loyal is the growth lever.

## B7. "Show me the top 3 products in EACH category." (Window function — interviewers test this)
```sql
WITH item_counts AS (
  SELECT category, item_purchased, COUNT(customer_id) AS total_orders,
         ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
  FROM customer GROUP BY category, item_purchased)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts WHERE item_rank <= 3 ORDER BY category, item_rank;
```
ANSWER (12 rows): Accessories — Jewelry 171, Belt 161, Sunglasses 161 | Clothing — Blouse 171, Pants 171, Shirt 169 | Footwear — Sandals 160, Shoes 150, Sneakers 145 | Outerwear — Jacket 163, Coat 161 (only 2 products in Outerwear)

## B8. "Which products do customers actually love?" (Rating, with a safety filter)
```sql
SELECT item_purchased, COUNT(customer_id) AS orders,
       ROUND(AVG(review_rating::numeric),2) AS avg_rating
FROM customer GROUP BY item_purchased
HAVING COUNT(customer_id) >= 50
ORDER BY avg_rating DESC LIMIT 5;
```
ANSWER: Gloves 3.86 (140 orders), Sandals 3.84 (160), Boots 3.82 (144), Hat 3.80 (154), Skirt 3.78 (158). Lowest: Shirt 3.62, Jeans 3.65, Blouse 3.68.
Takeaway: HAVING >= 50 orders stops a product with 2 lucky reviews looking like the best. Promote Gloves/Sandals/Boots; investigate Shirt quality/fit.

## B9. "Do customers who choose faster shipping spend more?"
```sql
SELECT shipping_type, COUNT(customer_id) AS orders,
       ROUND(AVG(purchase_amount),2) AS avg_spend, SUM(purchase_amount) AS revenue
FROM customer GROUP BY shipping_type ORDER BY avg_spend DESC;
```
ANSWER: 2-Day $60.73, Express $60.48, Free Shipping $60.41, Store Pickup $59.89, Next Day Air $58.63, Standard $58.46.
Takeaway: Gap is small (~$2.27 top to bottom). Premium-shipping users spend a little more, but shipping type is not a major revenue driver.

## B10. "Which season should we plan our biggest push for, and with what?" 
```sql
SELECT season, COUNT(customer_id) AS orders,
       ROUND(AVG(purchase_amount),2) AS avg_spend, SUM(purchase_amount) AS revenue
FROM customer GROUP BY season ORDER BY revenue DESC;
```
ANSWER: Fall $60,018 (avg $61.56 — highest), Spring $58,679, Winter $58,607, Summer $55,777 (lowest). Top category in EVERY season = Clothing (Fall $26,220; Spring $27,692; Summer $23,078; Winter $27,274).
Takeaway: Fall is the best season on both revenue and avg spend — plan the main campaign there, led by Clothing. Summer needs support.

## B11. "What is our biggest untapped opportunity?" ⭐ BEST BUSINESS QUESTION
```sql
SELECT subscription_status, COUNT(customer_id) AS repeat_buyers
FROM customer WHERE previous_purchases > 5
GROUP BY subscription_status;
```
ANSWER: Repeat buyers (>5 previous purchases) = 3,476 customers. Subscribed = 958 only (27.56%). NOT subscribed = 2,518 (72.44%).
Takeaway: 2,518 proven repeat buyers never subscribed. They already trust the store — they are the cheapest, warmest audience for a subscription campaign. This is a concrete, countable opportunity, not a vague suggestion.

## B12. "Which age group is most valuable?"
```sql
SELECT age_group, COUNT(customer_id) AS customers,
       ROUND(AVG(purchase_amount),2) AS avg_spend, SUM(purchase_amount) AS revenue
FROM customer GROUP BY age_group ORDER BY revenue DESC;
```
ANSWER (this is repo Q10): Young Adult — 1,028 customers, avg $60.45, revenue $62,143 (top). Middle-aged — 986, $60.04, $59,197. Adult — 942, $59.42, $55,978. Senior — 944, $59.07, $55,763.
Takeaway: Young Adults lead on total revenue AND per-order spend — priority segment for campaigns.

=====================================================================
HOW TO USE IN pgAdmin: Click database customer_behavior -> Query Tool -> paste ONE query -> F5.
HOW TO USE IN INTERVIEW: Question -> run/explain query (name the technique: CTE / window / CASE / HAVING / subquery) -> give the answer number -> 1-line takeaway. Never give a number without the takeaway.
EXTRA CHECK — repo Q2 answer: customers who used a discount AND spent >= average ($59.76) = 839 customers.
