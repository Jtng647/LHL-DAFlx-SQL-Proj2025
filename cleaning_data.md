**What issues will you address by cleaning the data?**

1 - Removing unnecessary columns that have no data in them (repeated for all columns with suspect all NULL entries)

2 - When a query uses the totaltransactionrevenu column, filter out all rows with NULL entries as they are inapplicable

3 - For queries that require a valid country entry, filter out NULL and '(not set)' entries

4 - When querying with the v2productname column, to acquire the product name, filter out NULL and '(not set)' entries

5 - Convert values from the unit_price column, the totaltransactionrevenue column, and the productprice column to intuitive dollar amounts (to 2 decimal places)


**Queries:**
**Below, provide the SQL queries you used to clean your data.**
------------------------------------------------------------------------------------

1 - Removing unnecessary columns that have no data in them - EXAMPLE
------------------------------------------------------------------------------------
	SELECT productrefundamount
	FROM all_sessions
	WHERE productrefundamount IS NOT NULL

'Since the above resulted in no data being shown'
	
 	DROP COLUMN productrefundamount

2 - When a query uses the totaltransactionrevenu column, filter out all rows with NULL entries as they are inapplicable
------------------------------------------------------------------------------------
	SELECT *
	FROM all_sessions
	WHERE totaltransactionrevenue IS NOT NULL

3 - For queries that require a valid country entry, filter out NULL and '(not set)' entries
------------------------------------------------------------------------------------
	SELECT *
	FROM all_sessions
	WHERE country != '(not set)'
		AND country IS NOT NULL

4 - When querying with the v2productname column, to acquire the product name, filter out NULL and '(not set)' entries
------------------------------------------------------------------------------------
	SELECT *
	FROM all_sessions
	WHERE v2productname != '(not set)'
		AND v2productname IS NOT NULL

5 - convert values from the unit_price column and the totaltransactionrevenue column to dollar amounts (to 2 decimal places)
------------------------------------------------------------------------------------
	SELECT
    	ROUND(s.totaltransactionrevenue/1000000, 2) AS totaltransactionrevenue,
		ROUND(s.productprice/1000000, 2) AS productprice,
    	ROUND(a.unit_price/1000000, 2) AS unit_price
	FROM all_sessions AS s

	JOIN analytics AS a
    		ON s.fullvisitorid = a.fullvisitorid

	GROUP BY s.totaltransactionrevenue, s.productprice, a.unit_price
