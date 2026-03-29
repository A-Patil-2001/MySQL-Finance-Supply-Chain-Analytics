# Finance Analysis

1) Gross Sales report: Monthly Product transaction
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
