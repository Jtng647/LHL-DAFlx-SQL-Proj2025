Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

    SELECT country, city,
        COUNT(totaltransactionrevenue) AS highest_transrev_level
    FROM all_sessions
    WHERE totaltransactionrevenue IS NOT NULL
    GROUP BY country, city
    ORDER BY highest_transrev_level DESC

Answer:

    Based on the resulting table from the query above, the United States is the country with the highest level of transaction revenues on this website. Among the specific (named) cities mentioned within the United States, San Francisco has the highest level of transaction revenues. It is important to note however, that while there is an entry with a higher transactional revenue level, the "city" in question is an assumed-amalgamation of all the data entries where the "city" information is not available to us in the demo dataset. Nevertheless, it does help reinforce the conclusion that the United Stated has the highest level of transactional revenues as a country. Thus, that data entry is still considered vaulable, and was kept for consideration.



**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:

To get AVG # of products ordered by COUNTRY
------------------------------------------------------------------------------
    WITH city_orders AS (
        SELECT country, city,
            COUNT(productsku) AS total_orders
        FROM all_sessions
        GROUP BY country, city
    )

    SELECT country, FLOOR(AVG(total_orders)) AS avg_orders_by_country
    FROM city_orders
    GROUP BY country
    ORDER BY avg_orders_by_country DESC
------------------------------------------------------------------------------

To get AVG # of products ordered by CITY
------------------------------------------------------------------------------
WITH visitor_specific_orders AS (
    SELECT city, visitid,
        COUNT(productsku) AS total_orders_per_visitor
    FROM all_sessions
    GROUP BY city, visitid
    )

    SELECT city, FLOOR(AVG(total_orders_per_visitor)) AS avg_orders_by_city
    FROM visitor_specific_orders
    GROUP BY city
    ORDER BY avg_orders_by_city DESC
------------------------------------------------------------------------------

Answer:

    The average number of products ordered from visitors in each country are shown in the resulting table from the corresponding query above. As an example, the top 10 countries with the highest average number of orders (rounded down to the nearest whole order) are:
    
    Country         |   Avg # of Products Ordered
    ---------------------------------------------
    United States   |           82
    United Kingdom  |           60
    Germany         |           56
    Italy           |           45
    Singapore       |           39
    India           |           37
    Canada          |           37
    Taiwan          |           34
    Israel          |           32
    Netherlands     |           31

    The average number of products ordered from visitors in each city are shown in the resulting table from the corresponding query above. As an example, the top 5 cities with the highest average number of orders (rounded down to the nearest whole order) are:
    
    City                |   Avg # of Products Ordered
    ---------------------------------------------
    Montreuil           |           2
    Avon                |           2
    Kharkiv             |           2
    South San Francisco |           1
    Sherbrooke          |           1

    The assumption was made that every visit (visitid) that was made with a product SKU (productsku) was an order made by that visitor. As such, a CTE was used to first acquire the COUNT of the orders (instances) made by that specific visitor ID. The information from that was then AVERAGED and GROUPED BY the corresponding city in the query to provide the average number of products ordered by those visitors, in their respective cities, including multiple visitors that were from the same city.
    
    Similarly, the same was done with the COUNT of the product SKUs per city in a CTE. The query then AVERAGED and GROUPED BY the corresponding country of those cities to return the average number of products ordered in that country.



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

    SELECT s.country, s.city, s.v2productcategory,
        COUNT(s.productsku) AS product_category_count,
        SUM(sbs.total_ordered) AS total_orders_by_SKU
    FROM all_sessions AS s

    JOIN sales_by_sku AS sbs
	    ON s.productsku = sbs.productsku

    WHERE s.country != '(not set)'
	    AND s.v2productcategory != '(not set)'
    GROUP BY s.country, s.city, s.v2productcategory
	    HAVING SUM(sbs.total_ordered) != 0
    ORDER BY country, product_category_count DESC, total_orders_by_SKU DESC

Answer:

    No, there does not seem to be a pattern that can be gleaned from the types/product categories ordered from visitors in each city and country. While we can certainly see which product types are the most popular, or most often ordered, for each country there does not seem to be any evident pattern or correlation to those categories. This could be due to factors like the needs, preferences, economical/financial situations or a multitude of aspects of the visitors from those countries. Additionally, it could also be a limitation of the types of products that the site offers, or the what visitors would consider ordering from as opposed to alternative vendors.

    Still, of note we can still see what is the most popular product category in a country across its cities. A few examples of this from the output are:

    - Australia - Most common category "Home/Shop by Brand/YouTube" is the most common across Melbourne, Sydney, and unaccounted for cities under the data that is "not available in demo dateset".
    - Canada - Both Toronto and Montreal both have "Men's T-Shirts" as their most common category
    
    If we alter the "GROUP BY" to take total_orders_by_SKU column as the highest sorting criteria, we also see that the United States lead in orders for most categories. The reasoning for this may vary (higher population, this site is more well known there, etc.) and is not included in the scope of this view, but nonetheless, it would be evident the United States ranks the highest in each product category's orders.




**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

    WITH ranked_orders AS (
        SELECT country, productsku, v2productname,
            COUNT(productsku) AS no_times_ordered,
            ROW_NUMBER() OVER (
			    PARTITION BY country
			    ORDER BY COUNT(productsku) DESC
			    ) AS rank
			
        FROM all_sessions
        WHERE country != '(not set)' AND productsku IS NOT NULL
        GROUP BY country, productsku, v2productname
	    )

	SELECT country, productsku, v2productname, no_times_ordered
	FROM ranked_orders

	WHERE rank = 1
	ORDER BY country

Answer:

    The top selling product from each city/country is shown in the output from the query above. While there are over a hundred entries listed due to the number of countries, some notable examples are:

    - Australia, with GGOEYDHJ056099 - 22 oz YouTube Bottle Infuser as its top-seller, ordered a total of 8 times
    - Canada, with GGOEYDHJ056099 - 22 oz YouTube Bottle Infuser as its top-seller, ordered a total of 13 times
    - China, with GGOEGAAX0339 - Google Men's Vintage Badge Tee White, ordered a total of 2 times
    - India, with GGOEYFKQ020699 - YouTube Custom Decals, ordered a total of 22 times
    - Indonesia, with GGOEYDHJ056099 - 22 oz YouTube Bottle Infuser as its top-seller, ordered a total of 6 times
    - New Zealand, with GGOEYDHJ056099 - 22 oz YouTube Bottle Infuser as its top-seller, ordered a total of 3 times
    - South Korea, with GGOEYFKQ020699 - YouTube Custom Decals, ordered a total of 2 times
    - United Kingdom, with GGOEYDHJ056099 - 22 oz YouTube Bottle Infuser as its top-seller, ordered a total of 22 times
    - United States, with GGOEGAAX0104 - Google Men's 100% Cotton Short Sleeve Hero Tee White, ordered a total of 163 times

    Some perceived patterns that seem to be worthy of note are:
    - That the "22 oz YouTube Bottle Infuser" seems to be the top selling product from several countries
    - The magnitude of the number of times ordered (no_times_ordered_) seems to correspond to the known population size of their respective countries
    - Some countries with the product only ordered once can be interpreted as there being a tie between the item listed, and other potential items also ordered from that country, but also only ordered once
    - This output reinforces our answer to question 1 and 2, in that the United States has the highest transaction revenue, and the highest average number of products ordered. It supports those conclusions by showing the United States' top selling product alone having been ordered 163 times, a much higher amount than the top selling item in any other country



**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

    SELECT country, city, v2productcategory,
        ROUND(SUM(totaltransactionrevenue)/1000000, 2) AS total_revenue

    FROM all_sessions

    WHERE country != '(not set)'
        AND totaltransactionrevenue > 0

    GROUP BY country, city, v2productcategory
    ORDER BY country, total_revenue DESC


Answer:

    There are multiple summarizations of impact that can be made based on the revenue generated from each city/country. One such impact is the market-country to which this website seemingly appeals to the most. This may inform the company which country they should focus their efforts on, or which country they may want to try expanding their market into.

    Based on the results generated by the query above, it is quite evident that the United States currently makes up a majority of the total revenue taken in by this website. The impact, then, is that this company's best-performing market is the United States. This can lead to business decisions such as stocking products that appeal the most to this country. Incidentally, including the product category in this query shows that the Nest, Electronics, and Bags product categories are the top 3 types of products ordered in the United States. So then this could mean stocking more variety of producs that fall into these categories. While the decisions that come out of it are still to be made by the business, the impact of the revenue generated by country/city is that it can provide direction for the business' future plans.






