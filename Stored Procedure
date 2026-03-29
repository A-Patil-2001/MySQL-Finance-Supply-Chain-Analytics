### 1) Create a store Procedure that can Determine the market badge based on the following logic, if total sold quantity > 5 million that market is considered as Goald else it is Silver.
- Input
  - market
  - fiscal year
 - Output
   - market badge 
``` sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_market_badge`(
		IN in_market VARCHAR(45),
        IN in_fiscal_year YEAR,
        OUT out_badge VARCHAR(45)
)
BEGIN
	DECLARE qty int default 0;

# Set default market to be India
	IF in_market = "" THEN
		SET in_market = "India";
	END IF;
    
# retrive total quantity for given market + fiscal_year
	SELECT 
		SUM(sold_quantity) into qty
	FROM fact_sales_monthly s
	JOIN dim_customer c
	ON s.customer_code = c.customer_code
	WHERE 
		get_fiscal_year(s.date) = in_fiscal_year AND
		c.market = in_market
	GROUP BY c.market; 
    
# Determini market badge
	IF qty > 5000000 THEN 	
		SET out_badge = "Gold";
	ELSE 
		SET out_badge = "Silver";
	END IF;
END
```

### 2) Create a store Procedure to get monthly Gross Sales for customers
- Input
  - Customer Code
``` sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_monthly_gross_sales_for_customer`(
		in_customer_code TEXT
)
BEGIN
	SELECT 
		s.date,
		ROUND(SUM(g.gross_price * s.sold_quantity),2) as gross_price_total
	FROM fact_sales_monthly s
	JOIN fact_gross_price g
	ON
		s.product_code = g.product_code AND
		g.fiscal_year = get_fiscal_year(s.date)
	WHERE 
		FIND_IN_SET(customer_code, in_customer_code) > 0
	GROUP BY s.date; 
END
```

### 3) Create a store Procedure for top n customers by Net Sales
- Input
  - Market
  - Fiscal Year
  - Top N
``` sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_customers_by_net_sales`(
		in_market VARCHAR(45),
        in_fiscal_year INT,
        in_top_n INT
)
BEGIN
	SELECT 
		c.customer,
		ROUND(SUM(n.net_sales)/1000000,2) as net_sales_mln
	FROM net_sales n
	JOIN dim_customer c
	ON n.customer_code = c.customer_code
	WHERE 
		n.fiscal_year = in_fiscal_year AND
        n.market = in_market
	GROUP BY c.customer
	ORDER BY net_sales_mln DESC
	LIMIT in_top_n;
END
```

### 4) Create a store Procedure for top n markets by Net Sales
- Input
  - Fiscal Year
  - Top N
``` sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_markets_by_net_sales`(
		in_fiscal_year INT,
        in_top_n INT
)
BEGIN
	SELECT 
		market,
		ROUND(SUM(net_sales)/1000000,2) as net_sales_mln
	FROM net_sales
	WHERE fiscal_year = in_fiscal_year
	GROUP BY market
	ORDER BY net_sales_mln DESC
	LIMIT in_top_n; 
END
```

### 5) Create a store Procedure for top n products by Net Sales
- Input
  - Fiscal Year
  - Top N
``` sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_products_by_net_sales`(
		in_fiscal_year INT,
        in_top_n INT
)
BEGIN
	SELECT 
		product,
		ROUND(SUM(net_sales)/1000000,2) as net_sales_mln
	FROM net_sales
	WHERE fiscal_year = in_fiscal_year
	GROUP BY product
	ORDER BY net_sales_mln DESC
	LIMIT in_top_n;
END
```

### 6) Create a store Procedure for getting Top n product for each devision by their Sold Quantity in giving financial year 2021
- Input
  - Fiscal Year
  - Top N
``` sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_per_devision_by_qty_sold`(
			in_fiscal_year INT,
            in_top_n INT
)
BEGIN
	WITH cte3 AS (SELECT 
						p.division,
						p.product,
						SUM(sold_quantity) as total_qty
					FROM fact_sales_monthly s
					JOIN dim_product p
						ON s.product_code = p.product_code
					WHERE s.fiscal_year = in_fiscal_year
					GROUP BY p.product, p.division),
		   cte4 AS (SELECT 
						*,
						DENSE_RANK() OVER(partition by division order by total_qty DESC) as drnk
					FROM cte3)
	SELECT * FROM cte4
	WHERE drnk <= in_top_n;
END
```

### 7) Create a store Procedure to calculate forecast accuracy 
- Input
  - Fiscal Year
``` sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_forecast_accuracy`(
		in_fiscal_year INT
)
BEGIN
	WITH forecast_err_table AS 
							(SELECT 
								e.customer_code,
								SUM(sold_quantity) as total_sold_quantity,
                                SUM(forecast_quantity) as total_forecast_quantity,
								SUM((forecast_quantity - sold_quantity)) as net_err,
								ROUND(SUM((forecast_quantity - sold_quantity))*100/SUM(forecast_quantity),2) as net_err_pct,
								SUM(ABS(forecast_quantity - sold_quantity)) as abs_err,
								ROUND(SUM(ABS(forecast_quantity - sold_quantity))*100/SUM(forecast_quantity),2) as abs_err_pct
							FROM fact_act_est e
							WHERE e.fiscal_year = in_fiscal_year
							GROUP BY customer_code) 
SELECT 
	e.customer_code,
	c.customer,
    c.market,
	e.total_sold_quantity,
    e.total_forecast_quantity,
    e.net_err,
    e.net_err_pct,
    e.abs_err,
    e.abs_err_pct,
    IF(abs_err_pct > 100, 0, 100 - abs_err_pct) as forecast_accuracy
FROM forecast_err_table e
JOIN dim_customer c
USING (customer_code)
ORDER BY forecast_accuracy DESC;
END
```
