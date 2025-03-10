**What are your risk areas? Identify and describe them.**

1 - Some cities may not be associated with the correct country, thus giving inaccurate resultes in queries requiring a country count, or city/country data driven conclusions.

2 - The totaltransactionrevenue should be a representation of REVENUE (money made) by the transaction, therefore it must be a POSITIVE NUMERIC VALUE.

3 - Both the tables "sales_by_sku" and "products" have columns that can be interpreted as showing the count of times an item has been ordered (sales_by_sku.total_ordered and products.orderedquantity, respectively). However this interpretation is an assumption, and needs to be confirmed. This can be checked 2 ways: by ensuring that the order counts correspond by productsku between the 2 tables, or that the sum totals of both columns are the same.

**QA Process:**
**Describe your QA process and include the SQL queries used to execute it.**
------------------------------------------------------------------------------------

1 - Identify country and all associated cities in data set > Identify geographically incorrect data > CASE function to correct
------------------------------------------------------------------------------------
'Identify country and all associated cities in data set'

	SELECT country, city
	FROM all_sessions
	WHERE country != '(not set)'
		AND city != 'not available in demo dataset'
		AND city != '(not set)'
	ORDER BY country

'Identify geographically incorrect data (example)'

	SELECT DISTINCT country, city
	FROM all_sessions
	WHERE country != '(not set)'
		AND country = 'Canada'
		AND city != 'not available in demo dataset'
		AND city != '(not set)'
	ORDER BY city

'The above query returns "New York" as city in Canada. To correct this:'

	SELECT DISTINCT
    		CASE
        		WHEN city = 'New York' THEN 'United States'
        		ELSE country
    		END AS country,
	city
	FROM all_sessions

	WHERE country != '(not set)'
		AND city != 'not available in demo dataset'
		AND city != '(not set)'
	ORDER BY country

2 - Confirm there are no negative or non-NUMERIC values in the totaltransactionrevenue column
------------------------------------------------------------------------------------
	SELECT totaltransactionrevenue
	FROM all_sessions
	WHERE totaltransactionrevenue IS NULL
    		OR totaltransactionrevenue <= 0
    		OR totaltransactionrevenue != totaltransactionrevenue::numeric

'The above query returns no negative results and no non-numeric results. However, an added WHERE clause to check for NULL values (which are non-numeric) results in very many NULLs. As such any query working with the totaltransactionrevenue column will need to include the WHERE clause below (as a data cleaning step):'

	WHERE totaltransactionrevenue IS NOT NULL

3 - Ensure that the order counts correspond by productsku between the 2 tables (sales_by_sku.total_ordered and products.orderedquantity) and that the sum totals are the same.
------------------------------------------------------------------------------------

'1st checking that the sum totals are the same'

	SELECT SUM(total_ordered) AS sbs_sum
	FROM sales_by_sku

'The above query returns "5524" '

	SELECT SUM(orderedquantity) AS prod_sum
	FROM products

'The above query returns "156733" '

"Since the sum totals are NOT the same, the differences by productsku needs to be checked.'

	SELECT sbs.productsku, sbs.total_ordered, p.sku, p.orderedquantity,
		CASE
			WHEN sbs.total_ordered = p.orderedquantity
				THEN 'Match'
			ELSE 'Does not match'
		END AS "QTY Match Status"
	FROM sales_by_sku AS sbs

	JOIN products AS p
		ON sbs.productsku = p.sku

'The above query returns an informative comparison. While all the sbs.productsku can be found and are JOINed ON p.sku, the sbs.total_ordered and p.orderedquantity do not match whatsoever. The only QTY Match Status with the "Match" output are when both tables return "0" for that productsku. Interestingly, it also looks like the quantities of p.orderedquantity > sbs.total_ordered.

Thus the conclusion can be inferred that these 2 columns do NOT show the same data, but rather p.orderedquantity shows the amount ordered as stock for the website. Conversely, sbs.total_ordered shows the amount of times the item has been ordered by a customer as a purchase. With this ascertained, a better understanding of this data can be achieved. For example, the ratio of certain items being ordered vs. stocked can show how well the item is performing from a sales perspective.'
