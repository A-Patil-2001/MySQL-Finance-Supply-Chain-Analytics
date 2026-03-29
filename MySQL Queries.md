# Finance Analysis

### 1) Gross Sales report:
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

### 2) As a product owner, I need aggregate monthly gross sales report for "Croma" India customer
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

### 3) Generate a yearly report for "Croma" India where there are two columns:
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
   
### 4) Monthly Gross Sales Report for customer "Croma" and Market "India":
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

### 5) Generate a report for Pre Invoice Discount %
``` sql
SELECT 
	s.date, 
    s.product_code,
    p.product, p.variant, 
    s.sold_quantity,
    ROUND(g.gross_price,2) as gross_price_per_item,
    ROUND(g.gross_price * s.sold_quantity,2) as gross_price_total,
    pre.pre_invoice_discount_pct
FROM fact_sales_monthly s
JOIN dim_product p
ON s.product_code = p.product_code
JOIN fact_gross_price g
ON 
	g.product_code = s.product_code AND
    g.fiscal_year = get_fiscal_year(s.date)
JOIN fact_pre_invoice_deductions pre
ON 	
	pre.customer_code = s.customer_code AND
    pre.fiscal_year = get_fiscal_year(s.date)
WHERE
	get_fiscal_year(s.date) = 2021
LIMIT 10000;
```

### 6)  Generate a report for Net Invoice Sales
``` sql
-- We have created a CTE (Common Table Expression), This is a tempervory table which is used only for the perticular query
 WITH cte1 AS (SELECT 
	s.date, 
    s.product_code,
    p.product, p.variant, 
    s.sold_quantity,
    ROUND(g.gross_price,2) as gross_price_per_item,
    ROUND(g.gross_price * s.sold_quantity,2) as gross_price_total,
    pre.pre_invoice_discount_pct
FROM fact_sales_monthly s
JOIN dim_product p
ON s.product_code = p.product_code
JOIN fact_gross_price g
ON 
	g.product_code = s.product_code AND
    g.fiscal_year = s.fiscal_year
JOIN fact_pre_invoice_deductions pre
ON 	
	pre.customer_code = s.customer_code AND
    pre.fiscal_year = s.fiscal_year) 
    
    SELECT 
	*,
    ROUND(gross_price_total - gross_price_total * pre_invoice_discount_pct,2) as net_invoice_sales
FROM cte1 ;
```

### 7) Generate a report for Post Invoice Decuction %
``` sql
SELECT 
	s.date, s.fiscal_year,
    s.customer_code, s.market,
    s.product_code, s.product, s.variant, 
    s.sold_quantity, s.gross_price_total, 
    ROUND(s.pre_invoice_discount_pct,2) as pre_invoice_discount_pct, 
    ROUND(s.gross_price_total - s.pre_invoice_discount_pct * s.gross_price_total, 2) as net_invoice_sales,
    ROUND((po.discounts_pct + po.other_deductions_pct),2) as post_invoice_discount_pct 
FROM sales_preinv_discount s
JOIN fact_post_invoice_deductions po
ON 
	s.customer_code = po.customer_code AND
    s.product_code = po.product_code AND
    s.date = po.date;
```

### 8) Generate a report for Net Sales
``` sql
SELECT
	*,
    ROUND((1-post_invoice_discount_pct) * net_invoice_sales,2) as net_sales
FROM sales_postinv_discount;
```

### 9) Generate a report for Top 5 markets by Net Sales for fiscal year 2021
- Columns required
  - market
  - net sales in millions
``` sql
SELECT 
	market,
    ROUND(SUM(net_sales)/1000000,2) as net_sales_mln
FROM net_sales
WHERE fiscal_year = 2021
GROUP BY market	
ORDER BY net_sales_mln DESC
LIMIT 5;
```

### 10) Generate a report for Top 5 customers by Net Sales for fiscal year 2021
- Columns required
  - customer
  - net sales in millions
``` sql
SELECT 
	c.customer,
    ROUND(SUM(n.net_sales)/1000000,2) as net_sales_mln
FROM net_sales n
JOIN dim_customer c
ON n.customer_code = c.customer_code
WHERE 
	n.fiscal_year = 2021 
GROUP BY c.customer
ORDER BY net_sales_mln DESC
LIMIT 5;
```

### 11) Generate a report for Top 5 products by Net Sales for fiscal year 2021
- Columns required
  - product
  - net sales in millions
``` sql
SELECT 
	product,
    ROUND(SUM(net_sales)/1000000,2) as net_sales_mln
FROM net_sales
WHERE fiscal_year = 2021
GROUP BY product
ORDER BY net_sales_mln DESC
LIMIT 5;
```
