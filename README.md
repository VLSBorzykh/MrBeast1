# 1)

SELECT cus.first_name, cus.last_name FROM customers cus

JOIN orders ord ON cus.customer_id = ord.customer_id

GROUP BY cus.first_name, cus.last_name, ord.order_date

HAVING COUNT(*) >= 2 AND ord.order_date BETWEEN '2023-07-17' AND '2023-10-17'

ORDER BY 1, 2

# 2)

SELECT AVG(ord.quantity) AS average, prd.category FROM orders ord

JOIN products prd ON prd.product_id = ord.product_id

WHERE price >= 50

GROUP BY prd.category

ORDER BY 2

# 3)

WITH sum_customers AS (
	
 SELECT ord.customer_id, SUM(prd.price) AS summa FROM orders ord

 JOIN products prd ON prd.product_id = ord.product_id
	
 GROUP BY ord.customer_id

), avg_sum AS (
	
 SELECT ord.order_id, ord.customer_id, ord.quantity, SUM(prd.price) AS summa FROM orders ord
	
 JOIN products prd ON prd.product_id = ord.product_id
	
 GROUP BY ord.order_id

)

SELECT cus.first_name, cus.last_name, cus.email FROM customers cus

JOIN sum_customers scus ON scus.customer_id = cus.customer_id

JOIN avg_sum asum ON asum.customer_id = cus.customer_id

GROUP BY cus.first_name, cus.last_name, cus.email, scus.summa

HAVING scus.summa > AVG(asum.summa)

# 4)

WITH sum_price AS (
	
SELECT ord.order_id, ord.customer_id, ord.product_id, ord.quantity, SUM(prd.price) AS summa FROM orders ord
	
JOIN products prd ON prd.product_id = ord.product_id
	
GROUP BY ord.order_id
	
HAVING SUM(prd.price) >= 1000

)

SELECT cus.first_name, cus.last_name FROM customers cus

JOIN sum_price spr ON spr.customer_id = cus.customer_id

JOIN products prd ON prd.product_id = spr.product_id

WHERE category != 'Electronics'

# 5)

DROP VIEW IF EXISTS v_avg_order;


CREATE VIEW v_avg_order AS (
	
WITH sum_order_id AS (
		
SELECT ord.order_id, ord.customer_id, ord.quantity, SUM(prd.price) AS summa FROM orders ord
		
JOIN products prd ON prd.product_id = ord.product_id
	
GROUP BY ord.order_id

), avg_sum_customer AS (

SELECT soid.customer_id, ROUND(AVG(soid.summa), 2) AS avg_summa FROM sum_order_id soid
	
GROUP BY soid.customer_id

), avg_all_sum AS (
		
SELECT ROUND(AVG(soid.summa), 2) AS avg_all FROM sum_order_id soid

)
	

SELECT cus.first_name, cus.last_name, ascus.avg_summa, (ascus.avg_summa - (SELECT avg_all FROM avg_all_sum)) AS difference FROM avg_sum_customer ascus

JOIN customers cus ON cus.customer_id = ascus.customer_id

JOIN sum_order_id soid ON soid.customer_id = ascus.customer_id

GROUP BY cus.first_name, cus.last_name, ascus.avg_summa

);

SELECT * FROM v_avg_order

# 6)

-- WORK IN PROGRESS


WITH customer_max_min AS (

SELECT ord.customer_id, MAX(ord.order_date) AS max_date, MIN(ord.order_date) AS min_date FROM orders ord

GROUP BY ord.customer_id

), range_max_min AS(

SELECT cusxn.customer_id, (cusxn.max_date - cusxn.min_date) AS range FROM customer_max_min cusxn

)


SELECT cus.first_name, cus.last_name, MAX(ord.order_date), MAX(ord.order_date), MAX(rgxn.range) FROM orders ord

JOIN range_max_min rgxn ON rgxn.customer_id = ord.customer_id

JOIN customers cus ON cus.customer_id = ord.customer_id

GROUP BY cus.first_name, cus.last_name

HAVING (SELECT MAX(rgxn.range) FROM range_max_min rgxn)
