# SQL_E-Commerce-Dataset
## ABOUT:
This project aims to **identify key patterns and trends** in the Google Merchandise Store, an e-commerce platform specializing in Google-branded products. We'll employ window functions, time formatting, and multiple common table expressions to clean and extract data that will inform our business analysis.
## DATASET
https://console.cloud.google.com/bigquery?ws=!1m5!1m4!4m3!1sbigquery-public-data!2sgoogle_analytics_sample!3sga_sessions_20170801
| Field Name | Data Type | Description |
| ------------- | ------------- | --- |
| fullVisitorId	| STRING	| The unique visitor ID.|
|date	|STRING	|The date of the session in YYYYMMDD format.|
|totals	|RECORD	|This section contains aggregate values across the session.|
|totals.bounces	|INTEGER	|Total bounces (for convenience). For a bounced session, the value is 1, otherwise it is null.|
|totals.hits	|INTEGER	|Total number of hits within the session.|
|totals.pageviews	|INTEGER	|Total number of pageviews within the session.|
|totals.visits	|INTEGER	|The number of sessions (for convenience). This value is 1 for sessions with interaction events. The value is null if there are no interaction events in the session.|
|totals.transactions	|INTEGER	|Total number of ecommerce transactions within the session.|
|trafficSource.source	|STRING	|The source of the traffic source. Could be the name of the search engine, the referring hostname, or a value of the utm_source URL parameter.|
|hits	|RECORD	|This row and nested fields are populated for any and all types of hits.|
|hits.eCommerceAction	|RECORD	|This section contains all of the ecommerce hits that occurred during the session. This is a repeated field and has an entry for each hit that was collected.|
|hits.eCommerceAction.action_type	|STRING	|"The action type. Click through of product lists = 1, Product detail views = 2, Add product(s) to cart = 3, Remove product(s) from cart = 4, Check out = 5, Completed purchase = 6, Refund of purchase = 7, Checkout options = 8, Unknown = 0. Usually this action type applies to all the products in a hit, with the following exception: when hits.product.isImpression = TRUE, the corresponding product is a product impression that is seen while the product action is taking place (i.e., a ""product in list view""). Example query to calculate number of products in list views: SELECT COUNT(hits.product.v2ProductName) FROM [foo-160803:123456789.ga_sessions_20170101] WHERE hits.product.isImpression == TRUE Example query to calculate number of products in detailed view: SELECT COUNT(hits.product.v2ProductName), FROM [foo-160803:123456789.ga_sessions_20170101] WHERE hits.ecommerceaction.action_type = '2' AND ( BOOLEAN(hits.product.isImpression) IS NULL OR BOOLEAN(hits.product.isImpression) == FALSE )"|
|hits.product	|RECORD	|This row and nested fields will be populated for each hit that contains Enhanced Ecommerce PRODUCT data.|
|hits.product.productQuantity	|INTEGER	|The quantity of the product purchased.|
|hits.product.productRevenue	|INTEGER	|The revenue of the product, expressed as the value passed to Analytics multiplied by 10^6 (e.g., 2.40 would be given as 2400000).|
|hits.product.productSKU	|STRING	|Product SKU.|
|hits.product.v2ProductName	|STRING	|Product Name.|
## Query 01: calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
```
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
|month	|visits	|pageviews	|transactions|
|---|---|---|---|
|201701	|64694	|257708	|713|
|201702	|62192	|233373	|733|
|201703	|69931	|259522	|993|

March 2017 shows a significant improvement in all metrics (visits, pageviews and transactions). 
## Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)
```
SELECT
    trafficSource.source as source,
    sum(totals.visits) as total_visits,
    sum(totals.Bounces) as total_no_of_bounces,
    (sum(totals.Bounces)/sum(totals.visits))* 100 as bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
GROUP BY source
ORDER BY total_visits DESC;
```
|source	|total_visits	|total_no_of_bounces	|bounce_rate|
|-|-|-|-|
|google	|38400	|19798	|51.56|
|(direct)	|19891	|8606	|43.27|
|youtube.com	|6351	|4238	|66.73|
|analytics.google.com	|1972	|1064	|53.96|
|Partners	|1788	|936	|52.35|
|m.facebook.com	|669	|430	|64.28|
|google.com	|368	|183	|49.73|
|dfa	|302	|124	|41.06|
|sites.google.com	|230	|97	|42.17|
|facebook.com	|191	|102	|53.4|
|reddit.com	|189	|54	|28.57|
|qiita.com	|146	|72	|49.32|
|quora.com	|140	|70	|50.0|
|baidu	|140	|84	|60.0|
|bing	|111	|54	|48.65|
|mail.google.com	|101	|25	|24.75|
|yahoo	|100	|41	|41.0|
|...|...|...|...|

The table analyzes the sources of website traffic, including total visits, total number of bounces, and bounce rate from various sources.
Optimize marketing strategies and improve user experience to reduce bounce rates, especially from sources with high bounce rates like Google and YouTube.
## Query 3: Revenue by traffic source by week, by month in June 2017
```
with 
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
  order by revenue DESC
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
  order by revenue DESC
)

select * from month_data
union all
select * from week_data;
```
|time_type	|month	|source	|revenue|
|-|-|-|-|
|Month	|201706	|groups.google.com	|101.96|
|Month	|201706	|yahoo	|20.39|
|Month	|201706	|search.myway.com	|105.939998|
|Month	|201706	|bing	|13.98|
|Week	|201726	|google	|5330.569964|
|Week	|201723	|search.myway.com	|105.939998|
|Week	|201724	|dfa	|2341.56|
|Week	|201725	|groups.google.com	|38.59|
|Week	|201725	|sites.google.com	|25.19|
|Week	|201725	|google.com	|23.99|
|Week	|201725	|google	|006.099991|
|Week	|201724	|dealspotr.com	|72.95|
|Week	|201722	|(direct)	|6888.899975|
|Week	|201722	|dfa	|1670.649998|
|Week	|201726	|groups.google.com	|63.37|
|...|...|...|...|

This table provides an overview of the revenue origins from different channels over periodic time frames. Revenue is primarily driven by direct sources and Google, with smaller but still significant contributions from other sources.
## Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.
```
with 
purchaser_data as(
  select
      format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
      (sum(totals.pageviews)/count(distinct fullvisitorid)) as avg_pageviews_purchase,
  from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
    ,unnest(hits) hits
    ,unnest(product) product
  where _table_suffix between '0601' and '0731'
  and totals.transactions>=1
  and product.productRevenue is not null
  group by month
),

non_purchaser_data as(
  select
      format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
      sum(totals.pageviews)/count(distinct fullvisitorid) as avg_pageviews_non_purchase,
  from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
      ,unnest(hits) hits
    ,unnest(product) product
  where _table_suffix between '0601' and '0731'
  and totals.transactions is null
  and product.productRevenue is null
  group by month
)

select
    pd.*,
    avg_pageviews_non_purchase
from purchaser_data pd
full join non_purchaser_data using(month)
order by pd.month;
```
|month	|avg_pageviews_purchase	|avg_pageviews_non_purchase|
|-|-|-|
|201706	|94.02050113895217	|316.86558846341671|
|201707	|124.23755186721992	|334.05655979568053|

The average pageviews for purchases increased from June to July, indicating that users who made purchases viewed more pages on average in July.
The average pageviews for non-purchases also increased from June to July, suggesting that users who did not make purchases were viewing more pages as well.
## Query 05: Average number of transactions per user that made a purchase in July 2017
```
select
    format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
    sum(totals.transactions)/count(distinct fullvisitorid) as Avg_total_transactions_per_user
from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    ,unnest (hits) hits,
    unnest(product) product
where  totals.transactions>=1
and product.productRevenue is not null
group by month;
```
|month	|Avg_total_transactions_per_user|
|-|-|
|201707	|4.16390041493776|

In July 2017, each user made an average of approximately 4.16 transactions
## Query 06: Average amount of money spent per session. Only include purchaser data in July 2017
```
select
    format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
    ((sum(product.productRevenue)/sum(totals.visits))/power(10,6)) as avg_revenue_by_user_per_visit
from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
  ,unnest(hits) hits
  ,unnest(product) product
where product.productRevenue is not null
and totals.transactions>=1
group by month;
```
|month	|avg_revenue_by_user_per_visit|
|-|-|
|201707	|43.856598348051243|

July 2017 had an average revenue per user per visit of 43.86. This insight can help the business evaluate and optimize marketing strategies to enhance engagement and conversion rates, turning visits into revenue more effectively.
## Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.
```
with buyer_list as(
    SELECT
        distinct fullVisitorId
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    , UNNEST(hits) AS hits
    , UNNEST(hits.product) as product
    WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
    AND totals.transactions>=1
    AND product.productRevenue is not null
)

SELECT
  product.v2ProductName AS other_purchased_products,
  SUM(product.productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
, UNNEST(hits) AS hits
, UNNEST(hits.product) as product
JOIN buyer_list using(fullVisitorId)
WHERE product.v2ProductName != "YouTube Men's Vintage Henley"
 and product.productRevenue is not null
GROUP BY other_purchased_products
ORDER BY quantity DESC;
```
|other_purchased_products	|quantity|
|-|-|
|Google Sunglasses	|20|
|Google Women's Vintage Hero Tee Black	|7|
|SPF-15 Slim & Slender Lip Balm	|6|
|Google Women's Short Sleeve Hero Tee Red Heather	|4|
|YouTube Men's Fleece Hoodie Black	|3|
|Google Men's Short Sleeve Badge Tee Charcoal	|3|
|Android Wool Heather Cap Heather/Black	|2|
|...|...|

Google Sunglasses was the most frequently purchased product, with a quantity of 20. 

Google Women's Vintage Hero Tee Black and SPF-15 Slim & Slender Lip Balm were also popular, with quantities of 7 and 6 respectively.

Various other products were purchased in smaller quantities, ranging from 4 to 1 units.

This suggests that customers who bought the YouTube Men's Vintage Henley in July 2017 also showed a diverse interest in other Google and YouTube merchandise, indicating potential cross-selling opportunities.
## Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase. Add_to_cart_rate = number product  add to cart/number product view. Purchase_rate = number product purchase/number product view. The output should be calculated in product level.
```
with
product_view as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_product_view
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '2'
  GROUP BY 1
),

add_to_cart as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_addtocart
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '3'
  GROUP BY 1
),

purchase as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '6'
  and product.productRevenue is not null   --phải thêm điều kiện này để đảm bảo có revenue
  group by 1
)

select
    pv.*,
    num_addtocart,
    num_purchase,
    round(num_addtocart*100/num_product_view,2) as add_to_cart_rate,
    round(num_purchase*100/num_product_view,2) as purchase_rate
from product_view pv
left join add_to_cart a on pv.month = a.month
left join purchase p on pv.month = p.month
order by pv.month;
```
|month	|num_product_view	|num_addtocart	|num_purchase	|add_to_cart_rate	|purchase_rate|
|-|-|-|-|-|-|
|201701	|25787	|7342	|2143	|28.47	|8.31|
|201702	|21489	|7360	|2060	|34.25	|9.59|
|201703	|23549	|8782	|2977	|37.29	|12.64|

Overall, the data without August 2017 suggests a positive trend in user engagement and conversion rates, with significant increases in product views, items added to the cart, and purchases from January to July 2017. The exclusion of the incomplete data for August ensures the integrity and accuracy of these insights

