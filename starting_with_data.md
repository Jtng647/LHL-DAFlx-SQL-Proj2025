**Question 1:** 
    
    Which visitors have visited the website the most times, and where are they from?

SQL Queries:

    SELECT country, city, fullVisitorId,
        COUNT(visitid) AS visit_count
    FROM all_sessions

    WHERE fullVisitorId IS NOT NULL

    GROUP BY  fullVisitorId, city, country
	    HAVING COUNT(visitid) > 1
    ORDER BY visit_count DESC

Answer: 

    From the result of the query above, visitor ID: 3608475193341679870 from the United States, has visited this site the most often, at 10 visits. The top 10 visitors with multiple visits to this site are all also from the United States.

--------------------------------------------------------------------------------------------------------------------------------

**Question 2:**

    Which product item is the most popular, and will need to be restocked earlier?

SQL Queries:

    SELECT sbs.productSKU, sr.productname, sr.restockingLeadTime,
        SUM(sbs.total_ordered) AS total_orders
    FROM sales_by_sku AS sbs

    JOIN sales_report AS sr
	    ON sbs.productSKU = sr.productSKU
	
    GROUP BY sbs.productSKU, sr.productname, sr.restockingLeadTime
    ORDER BY total_orders DESC, sr.restockingLeadTime DESC
    LIMIT 1

Answer:

    Based on the result from the query above, the Ballpoint LED Light Pen (SKU: GGOEGOAQ012899) is the most popular item, having been ordered 456 times. In addition to its popularity, due to its long lead time of 11 (days), it should be restocked earlier than other products.

--------------------------------------------------------------------------------------------------------------------------------

**Question 3:** 

    Which "Tech Company" branded product (Android, YouTube, Google, Waze, Nest) is the most ordered? Which "Tech Company" has the most items in the top 5?

SQL Queries:

    SELECT 
        CASE
            WHEN s.v2productname LIKE '%Android%' THEN 'Android'
            WHEN s.v2productname LIKE '%YouTube%' THEN 'YouTube'
            WHEN s.v2productname LIKE '%Google%' THEN 'Google'
            WHEN s.v2productname LIKE '%Waze%' THEN 'Waze'
            WHEN s.v2productname LIKE '%Nest%' THEN 'Nest'
        END AS brand,
        s.v2productname AS brand_name,
        COUNT(sbs.total_ordered) AS total_items_ordered
    FROM sales_by_sku AS sbs

    JOIN sales_report AS sr
	    ON sbs.productSKU = sr.productSKU
    JOIN all_sessions AS s
	    ON s.productsku = sbs.productSKU

    WHERE
        (s.v2productname LIKE '%Android%' OR
        s.v2productname LIKE '%YouTube%' OR
        s.v2productname LIKE '%Google%' OR
        s.v2productname LIKE '%Waze%' OR
        s.v2productname LIKE '%Nest%')
    GROUP BY brand, s.v2productname
    ORDER BY total_items_ordered DESC

Answer:

    Based on the results of the query above, Google's Men's 100% Cotton Short Sleeve Hero Tee White looks to be the best seller out of all the Tech Company branded items, with an order count of 590 orders.
    
    However, within the top 5 Tech Company branded items, YouTube as a brand is the most commonly ordered. It holds spots 2 through 5 of the top 5 items: "22 oz YouTube Bottle Infuser" (490 orders), "YouTube Twill Cap" (442 orders), "YouTube Custom Decals" (422 orders), and "YouTube Men's Short Sleeve Hero Tee Black" (394 orders), respectively.