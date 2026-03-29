# Supply Chain Analysis

### 1) Create Forecast Accuracy Report using Common tebale expression (CTE)
``` sql
WITH forecast_err_table AS (SELECT 
								e.customer_code,
								SUM(sold_quantity) as total_sold_quantity,
                                SUM(forecast_quantity) as total_forecast_quantity,
								SUM((forecast_quantity - sold_quantity)) as net_err,
								ROUND(SUM((forecast_quantity - sold_quantity))*100/SUM(forecast_quantity),2) as net_err_pct,
								SUM(ABS(forecast_quantity - sold_quantity)) as abs_err,
								ROUND(SUM(ABS(forecast_quantity - sold_quantity))*100/SUM(forecast_quantity),2) as abs_err_pct
							FROM fact_act_est e
							WHERE e.fiscal_year = 2021
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
```

### 2) Create Forecast Accuracy Report using Temporary Table
``` sql
CREATE TEMPORARY TABLE forecast_err_table
SELECT 
	e.customer_code,
	SUM(sold_quantity) as total_sold_quantity,
	SUM(forecast_quantity) as total_forecast_quantity,
	SUM((forecast_quantity - sold_quantity)) as net_err,
	ROUND(SUM((forecast_quantity - sold_quantity))*100/SUM(forecast_quantity),2) as net_err_pct,
	SUM(ABS(forecast_quantity - sold_quantity)) as abs_err,
	ROUND(SUM(ABS(forecast_quantity - sold_quantity))*100/SUM(forecast_quantity),2) as abs_err_pct
FROM fact_act_est e
WHERE e.fiscal_year = 2021
GROUP BY customer_code;

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
```
