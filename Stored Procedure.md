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
