# Finance Analysis

1) Gross Sales report:
- Monthly Product transaction
     - column required
       - Month
       - Product Name
       - Variant
       - Sold Quantity
       - Gross price per item
       - Gross price total
``` sql
SELECT 
	s.date, s.product_code,
    p.product, p.variant, s.sold_quantity,
    ROUND(g.gross_price,2),
    ROUND(g.gross_price * s.sold_quantity,2) as gross_price_total
FROM fact_sales_monthly s
JOIN dim_product p
ON s.product_code = p.product_code
JOIN fact_gross_price g
ON 
	g.product_code = s.product_code AND
    g.fiscal_year = get_fiscal_year(s.date)
WHERE
	customer_code = 90002002 AND
	get_fiscal_year(date) = 2021
ORDER BY date ASC
LIMIT 10000;
```

2) As a product owner, I need aggregate monthly gross sales report for "Croma" India customer
   - Report should have the following fields
     - Month
     - Total Gross Sales amount for "Croma" India
``` sql
SELECT 
    s.date AS month,
    ROUND(SUM(g.gross_price * s.sold_quantity),2) as gross_price_total
FROM fact_sales_monthly s
JOIN fact_gross_price g
ON
	s.product_code = g.product_code AND
    g.fiscal_year = get_fiscal_year(s.date)
WHERE customer_code = 90002002
GROUP BY s.date
``` 

3) Generate a yearly report for "Croma" India where there are two columns
   - Fiscal year
   - Total Gross Sales amount in the year from "Croma"
``` sql
SELECT 
    get_fiscal_year(date) as fiscal_year,
    ROUND(SUM(g.gross_price * s.sold_quantity),2) as gross_price_total
FROM fact_sales_monthly s
JOIN fact_gross_price g
ON
	s.product_code = g.product_code AND
    g.fiscal_year = get_fiscal_year(s.date)
WHERE 
	customer_code = 90002002
GROUP BY get_fiscal_year(date)
ORDER BY fiscal_year ASC;
```
   
4) Monthly Gross Sales Report for customer "Croma" and Market "India":
   - Cloumn Required
     - Date
     - Monthly Sales
``` sql
SELECT 
		s.date,
		ROUND(SUM(g.gross_price * s.sold_quantity),2) as gross_price_total
	FROM fact_sales_monthly s
	JOIN fact_gross_price g
	ON
		s.product_code = g.product_code AND
		g.fiscal_year = get_fiscal_year(s.date)
	WHERE 
		customer_code IN (90002016, 90002008)
	GROUP BY s.date; 
    
    SELECT * FROM dim_customer
    WHERE customer LIKE "%amazon%" AND market = "India";
    
    SELECT FIND_IN_SET(9000208, "90002016,90002008");
```
