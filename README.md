## **I. INTRODUCTION**
In this project, **_business performance of an e-commerce website_** was analyzed using **_SQL_** in **_Google BigQuery_**, with some advanced SQL techniques like **_window function_** and **_aggregate function_**. The analysis covered critical business aspects such as **_website traffic, customer behavior patterns, revenue generation, sales performance,_** and **_conversion optimization_**. The **_insights_** derived from this work provided **_support to the Sales and Marketing teams_**, enabling them to make informed, data-driven decisions.
## **II. DATASET**
- The e-commerce dataset is stored in a public Google BigQuery dataset under ID: `bigquery-public-data.google_analytics_sample.ga_sessions`.
  
- Table Schema: https://support.google.com/analytics/answer/3437719?hl=en
  
## **III. KEY TO ANALYZE
  - **_Website Traffic & User Behavior:_** Analyze traffic patterns (`total visits, pageviews`), user engagement (`bounce rate`), and how different types of users (purchasers vs. non-purchasers) behave in terms of pageviews.

  - **_Sales Performance & Revenue Analysis:_** Evaluate the `revenue generated` from different traffic sources over time (by week and month).
    
  - **_Product Performance & Conversion Optimization_**: Analyze `related products purchased by customers` (to identify product affinity and potential cross-selling opportunities) and calculate `cohort map` from product view to addtocart to purchasec (to identify conversion rates and shopping behavior).
## **IV. EXPLORE THE DATASET**
### **Query 1: Calculate total visit, pageview, transaction for Jan, Feb and March 2017**
```SQL
SELECT
  format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
  SUM(totals.visits) AS visits,
  SUM(totals.pageviews) AS pageviews,
  SUM(totals.transactions) AS transactions,
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _TABLE_SUFFIX BETWEEN '0101' AND '0331'
GROUP BY 1
ORDER BY 1;
```
| month	| visits	| pageviews	| transactions| 
| --- | --- | --- | --- |
|201701	|64694	|257708	|713|
| 201702	| 62192	| 233373	| 733| 
| 201703	| 69931	| 259522	| 993| 



### **Query 2: Bounce rate per traffic source in July 2017**
- Bounce_rate = num_bounce/total_visit
```SQL
WITH cal_total AS (
  SELECT
      trafficSource.source as source,
      sum(totals.visits) as total_visits,
      sum(totals.Bounces) as total_no_of_bounces,
      (sum(totals.Bounces)/sum(totals.visits))* 100.00 as bounce_rate
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
  GROUP BY source
  ORDER BY total_visits DESC
)
SELECT
  source
  ,total_visits
  ,total_no_of_bounces
  ,round(100.0*(total_no_of_bounces/total_visits),3) bounce_rate
FROM cal_total
GROUP BY
  source
  ,total_visits 
  ,total_no_of_bounces
ORDER BY total_visits desc;
```
| source	| total_visits	| total_no_of_bounces	| bounce_rate| 
| --- | --- | --- | --- |
| google	| 38400	| 19798	| 51.557| 
| (direct)	| 19891	| 8606	| 43.266| 
| youtube.com	| 6351	| 4238	| 66.73| 
| analytics.google.com	| 1972	| 1064	| 53.955| 
| Partners	| 1788	| 936	| 52.349| 
| m.facebook.com	| 669	| 430	| 64.275| 
| google.com	| 368	| 183	| 49.728| 
| dfa	| 302	| 124	| 41.06| 
| sites.google.com	| 230	| 97	| 42.174| 
| facebook.com	| 191	| 102	| 53.403| 
| reddit.com	| 189	| 54	| 28.571| 
| qiita.com	| 146	| 72	| 49.315| 


### **Query 3: Revenue by traffic source by week, by month in June 2017**
```SQL
WITH 
month_data as(
  SELECT
    "Month" as time_type,
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    trafficSource.source AS source,
    SUM(p.productRevenue)/1000000 AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    unnest(hits) hits,
    unnest(product) p
  WHERE p.productRevenue is not null
  GROUP BY 1,2,3
  ORDER BY revenue DESC
),

week_data as(
  SELECT
    "Week" as time_type,
    format_date("%Y%W", parse_date("%Y%m%d", date)) as week,
    trafficSource.source AS source,
    SUM(p.productRevenue)/1000000 AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    unnest(hits) hits,
    unnest(product) p
  WHERE p.productRevenue is not null
  GROUP BY 1,2,3
  ORDER BY revenue DESC
)

SELECT * FROM month_data
UNION ALL
SELECT * from week_data
ORDER BY time_type
```
| time_type	| time	| source	| revenue| 
| --- | --- | --- | --- |
| Month	| 201706	| (direct)	| 97333.6197| 
| Month	| 201706	| google	| 18757.1799| 
| Month	| 201706	| dfa	| 8862.23| 
| Month| 	201706|	mail.google.com| 	2563.13| 
| Month	| 201706| 	search.myway.com| 	105.94| 
| Month| 	201706| 	groups.google.com| 	101.96| 
| Month| 	201706| 	chat.google.com| 	74.03| 
| Month| 	201706| 	dealspotr.com| 	72.95| 
| Month| 	201706| 	mail.aol.com| 	64.85| 
| Month| 	201706| 	phandroid.com| 	52.95| 
| Month| 	201706| 	sites.google.com| 	39.17| 
| Month| 	201706| 	google.com| 	23.99| 
| Month| 	201706| 	yahoo| 	20.39| 
| Month| 	201706| 	youtube.com| 	16.99| 
| Month| 	201706| 	bing| 	13.98| 
| Month	| 201706| 	l.facebook.com| 	12.48| 
| Week	| 201722	| (direct)	| 6888.9| 
| Week	| 201722	| google| 	2119.39| 
| Week	| 201722	| dfa	| 1670.65| 
| Week	| 201722	| sites.google.com	| 13.98| 
| Week	| 201723	| (direct)	| 17325.6799| 
| Week| 	201723	| dfa| 	1145.28| 
| Week| 	201723	| google	| 1083.95| 
| Week	| 201723| 	search.myway.com| 	105.94| 
| Week| 	201723	| chat.google.com	| 74.03| 
| Week	| 201723	| youtube.com	| 16.99| 
| Week	| 201724	| (direct)	| 30908.9099| 
| Week	| 201724	| google| 	9217.17| 
| Week	| 201724| 	mail.google.com	| 2486.86| 
| Week	| 201724	| dfa	| 2341.56| 
| Week	| 201724	| dealspotr.com	| 72.95| 
| Week	| 201724	| bing	| 13.98| 
| Week| 	201724	| l.facebook.com	| 12.48| 
| Week	| 201725	| (direct)	| 27295.3199| 
| Week	| 201725	| google	| 1006.1| 
| Week	| 201725| 	mail.google.com	| 76.27| 
| Week	| 201725| 	mail.aol.com| 	64.85| 
| Week	| 201725	| phandroid.com	| 52.95| 
| Week	| 201725	| groups.google.com	| 38.59| 
| Week	| 201725	| sites.google.com	| 25.19| 
| Week	| 201725	| google.com| 	23.99| 
| Week	| 201726	| (direct)	| 14914.81| 
| Week	| 201726	| google	| 5330.57| 
| Week	| 201726	| dfa	|3704.74| 
| Week	| 201726	| groups.google.com	| 63.37| 
| Week	| 201726	| yahoo	| 20.39| 


### **Query 4: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017**
```SQL
WITH 
purchaser_data AS (
  SELECT
      format_date("%Y%m",parse_date("%Y%m%d",date)) AS month,
      (sum(totals.pageviews)/count(distinct fullvisitorid)) AS avg_pageviews_purchase,
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
    ,unnest(hits) hits
    ,unnest(product) product
  WHERE _table_suffix between '0601' and '0731'
    AND totals.transactions>=1
    AND product.productRevenue is not null
  GROUP BY month
),

non_purchaser_data AS (
  SELECT
      format_date("%Y%m",parse_date("%Y%m%d",date)) AS month,
      sum(totals.pageviews)/count(distinct fullvisitorid) AS avg_pageviews_non_purchase,
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
      ,unnest(hits) hits
    ,unnest(product) product
  WHERE _table_suffix between '0601' and '0731'
    AND totals.transactions is null
    AND product.productRevenue is null
  GROUP BY month
)

SELECT
    pd.*,
    avg_pageviews_non_purchase
FROM purchaser_data pd
FULL JOIN non_purchaser_data using(month)
ORDER BY pd.month;
```
| month	| avg_pageviews_purchase| avg_pageviews_non_purchase
| --- | --- | --- |
| 201706	| 94.02050113895217	| 316.86558846341671| 
| 201707	| 124.23755186721992	| 334.05655979568053| 


### **Query 5: Average number of transactions per user that made a purchase in July 2017**
```SQL
SELECT  
   format_date('%Y%m', cast(date as date format 'YYYYMMDD')) month
   ,sum(totals.transactions)/count(distinct fullVisitorId) Avg_total_transactions_per_user
FROM
    `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,    --chỉnh lại lấy 201707*
    UNNEST(hits) hits,
    UNNEST(hits.product) product
WHERE 
    product.productRevenue IS NOT NULL
    AND totals.transactions >= 1
    --AND extract(month from cast(date as date format 'YYYYMMDD')) = 07
GROUP BY month;
```
| month	| avg_total_transactions_per_user| 
| --- | --- |
| 201707	| 4.16390041493776| 



### **Query 6: Average amount of money spent per session. Only include purchaser data in July 2017**
```SQL
SELECT  
  format_date('%Y%m', cast(date as date format 'YYYYMMDD')) month
  ,round((sum(product.productRevenue)/1000000)/count(totals.visits),2) as avg_revenue_by_user_per_visit
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
      UNNEST(hits) hits,
      UNNEST(hits.product) product
WHERE 
  product.productRevenue is not null
  and totals.transactions is not null
  and extract(month from cast(date as date format 'YYYYMMDD')) = 07
GROUP BY month;
```
| month	| avg_revenue_by_user_per_visit| 
| --- | --- |
| 201707	| 43.86| 



### **Query 7: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017**
```SQL
WITH v1 AS (
  SELECT
    distinct fullVisitorId
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
        UNNEST(hits) hits,
        UNNEST(hits.product) product
  WHERE product.productRevenue IS NOT NULL 
        AND extract(month from cast(date as date format 'YYYYMMDD')) = 07
        AND product.v2ProductName = "YouTube Men's Vintage Henley"
),
purchased AS (
  SELECT
    fullVisitorId
    ,product.v2ProductName name
    ,product.productQuantity quantity
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
              UNNEST(hits) hits,
              UNNEST(hits.product) product
  WHERE product.productRevenue is not null 
    AND extract(month from cast(date as date format 'YYYYMMDD')) = 07
)
SELECT
  p.name other_purchased_products
  ,sum(p.quantity) quantity
FROM v1 LEFT JOIN purchased p
ON v1.fullVisitorId = p.fullVisitorId
WHERE p.name != "YouTube Men's Vintage Henley"
GROUP BY p.name
ODER BY quantity desc;
```
| other_purchased_products	| quantity| 
| --- | --- |
| Google Sunglasses	| 20| 
| Google Women's Vintage Hero Tee Black	| 7| 
| SPF-15 Slim & Slender Lip Balm	| 6| 
| Google Women's Short Sleeve Hero Tee Red Heather	| 4| 
| Google Men's Short Sleeve Badge Tee Charcoal	| 3| 
| YouTube Men's Fleece Hoodie Black	| 3| 
| YouTube Twill Cap	| 2| 
| Google Men's Short Sleeve Hero Tee Charcoal	| 2| 
| Red Shine 15 oz Mug	| 2| 
| Crunch Noise Dog Toy	| 2| 


### **Query 8: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. Output calculated in product level**
- Add_to_cart_rate = number product add to cart / number product view.
- Purchase_rate = number product purchase/number product view.

```SQL
WITH total_cohort AS (
  SELECT
    format_date('%Y%m', cast(date as date format 'YYYYMMDD')) month
    ,SUM(CASE WHEN eCommerceAction.action_type = '2' THEN 1 ELSE 0 END) num_product_view
    ,SUM(CASE WHEN eCommerceAction.action_type = '3' THEN 1 ELSE 0 END) num_addtocart
    ,SUM(CASE WHEN eCommerceAction.action_type = '6' 
              AND product.productRevenue IS NOT NULL 
              THEN 1 ELSE 0 END) num_purchase
  FROM
      `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
      UNNEST(hits) hits,
      UNNEST(hits.product) product
  WHERE EXTRACT(month from cast(date as date format 'YYYYMMDD')) IN (1,2,3)
  GROPU BY month
)
SELECT
  month
  ,num_product_view
  ,num_addtocart
  ,num_purchase
  ,round(100.0*num_addtocart/num_product_view,2) add_to_cart_rate
  ,round(100.0*num_purchase/num_product_view,2) purchase_rate
FROM total_cohort
ORDER BY month;
```
| month	| num_product_view	| num_addtocart	| num_purchase	| add_to_cart_rate	| purchase_rate| 
| --- | --- | --- | --- | --- | --- |
| 201701	| 25787	| 7342	| 2143	| 28.47	| 8.31| 
| 201702	| 21489	| 7360	| 2060	| 34.25	| 9.59| 
| 201703	| 23549	| 8782	| 2977	| 37.29	| 12.64| 



## **V. CONCLUSION**
In conclusion, my analysis of the eCommerce dataset using SQL on Google BigQuery has uncovered valuable **_insights into total visits, pageviews, transactions, bounce rates, and revenue per traffic source_**, which can inform future business decisions. The **_next step_** will involve **_visualizing_** these insights and key trends using software such as Power BI or Tableau. Overall, this project demonstrates the **_effectiveness of employing SQL and big data tools_** like Google BigQuery to **_derive meaningful insights from large datasets_**.
