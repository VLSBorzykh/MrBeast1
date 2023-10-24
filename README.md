# Задание 1
```
SELECT Customers.customer_id, Customers.first_name, Customers.last_name
FROM Customers WHERE ( SELECT COUNT(Orders.order_id) FROM Orders
WHERE Orders.customer_id = Customers.customer_id
AND Orders.order_date >= current_date - interval '3 months' ) > 2;
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/fd3295ec-c273-4052-8476-8f71cb598f33)


# Задание 2
```
SELECT Products.category, AVG(Orders.quantity)
AS average_order_size FROM Products JOIN Orders
ON Products.product_id = Orders.product_id
WHERE Products.price >= 50 GROUP BY Products.category;
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/0cd515d1-5b44-4e90-8f42-40436982ba14)


# Задание 3
```
WITH AverageOrderCost AS (
    SELECT AVG(p.price * o.quantity) AS avg_order_cost
    FROM Orders o
    JOIN Products p ON o.product_id = p.product_id
)
SELECT c.customer_id, c.first_name, c.last_name, SUM(p.price * o.quantity) AS total_order_cost
FROM Customers c
JOIN Orders o ON c.customer_id = o.customer_id
JOIN Products p ON o.product_id = p.product_id
GROUP BY c.customer_id, c.first_name, c.last_name
HAVING SUM(p.price * o.quantity) > (SELECT avg_order_cost FROM AverageOrderCost)
ORDER BY total_order_cost DESC;
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/3bdd2d37-b3f6-4b9b-8932-638c9d044f54)


# Задание 4
```
SELECT c.customer_id, c.first_name, c.last_name
FROM Customers c
WHERE c.customer_id IN (
    SELECT o.customer_id
    FROM Orders o
    JOIN Products p ON o.product_id = p.product_id
    WHERE p.category <> 'Electronics'
    GROUP BY o.customer_id
    HAVING SUM(p.price * o.quantity) > 1000
);
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/46bfcf55-3edd-45e6-99c6-389037fd105f)


# Задание 5
```
CREATE OR REPLACE VIEW CustomerOrderStats AS
SELECT
    Customers.customer_id,
    Customers.first_name,
    Customers.last_name,
    AVG(Products.price * Orders.quantity) AS avg_order_cost,
    AVG(Products.price * Orders.quantity) - (
        SELECT AVG(Products.price * Orders.quantity)
        FROM Orders
        JOIN Products ON Orders.product_id = Products.product_id
    ) AS deviation
FROM Customers
LEFT JOIN Orders ON Customers.customer_id = Orders.customer_id
LEFT JOIN Products ON Orders.product_id = Products.product_id
GROUP BY Customers.customer_id, Customers.first_name, Customers.last_name;
```
# Задание 6
```
SELECT C.customer_id, C.first_name, C.last_name, MAX(date_diff) AS max_diff
FROM Customers C
JOIN (
    SELECT O1.customer_id, 
           MAX(O1.order_date - O2.order_date) AS date_diff
    FROM Orders O1
    JOIN Orders O2 ON O1.customer_id = O2.customer_id
    WHERE O1.order_date > O2.order_date
    GROUP BY O1.customer_id
) AS max_diffs ON C.customer_id = max_diffs.customer_id
GROUP BY C.customer_id, C.first_name, C.last_name
ORDER BY max_diff DESC
LIMIT 1;
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/a8c369a2-948a-49bf-96cc-dc2330200ec4)


# Задание 7
```
WITH RecentOrderTotals AS (
    SELECT
        o.customer_id,
        SUM(p.price * o.quantity) AS total_cost
    FROM Orders o
    JOIN Products p ON o.product_id = p.product_id
    WHERE o.order_date >= (CURRENT_DATE - INTERVAL '2 months')
    GROUP BY o.customer_id
),
PreviousOrderTotals AS (
    SELECT
        o.customer_id,
        SUM(p.price * o.quantity) AS total_cost
    FROM Orders o
    JOIN Products p ON o.product_id = p.product_id
    WHERE o.order_date >= (CURRENT_DATE - INTERVAL '4 months')
        AND o.order_date < (CURRENT_DATE - INTERVAL '2 months')
    GROUP BY o.customer_id
)
SELECT c.customer_id, c.first_name, c.last_name
FROM Customers c
JOIN RecentOrderTotals recent ON c.customer_id = recent.customer_id
LEFT JOIN PreviousOrderTotals previous ON c.customer_id = previous.customer_id
WHERE recent.total_cost > COALESCE(previous.total_cost, 0);
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/a7dcc840-5e15-4155-8ffb-77a509a0864b)


# Задание 8
```
UPDATE Products
SET price = price * 10
WHERE category = 'Clothing';
```
# Задание 9
```
WITH CategoryAvgPrice AS (
    SELECT
        category,
        AVG(price) AS avg_price
    FROM Products
    GROUP BY category
)
SELECT category, avg_price
FROM CategoryAvgPrice
WHERE avg_price = (SELECT MAX(avg_price) FROM CategoryAvgPrice)
    OR avg_price = (SELECT MIN(avg_price) FROM CategoryAvgPrice);
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/3458bfd6-ca4d-437c-b434-3ffc36b7462e)

# задание 10
```
DELETE FROM Orders
WHERE quantity > (
    SELECT AVG(quantity)
    FROM Orders
);
```

# # 24/10

# 1
```
SELECT p.id AS p_id, COUNT(pv.id) AS count_of_visits FROM person p
JOIN person_visits pv ON pv.person_id = p.id
GROUP BY 1
ORDER BY p_id ASC, count_of_visits DESC
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/30090502-96c3-4e44-aec5-562dada22310)

# 2
```
SELECT p.name, COUNT(pv.id) AS count_of_visits FROM person p
JOIN person_visits pv ON pv.person_id = p.id
GROUP BY 1
ORDER BY count_of_visits DESC
LIMIT 4
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/ac244ee2-fc9b-4b49-a469-5c461e8f6ab1)

# 3
```
WITH fav_rest_visit AS (
	SELECT pzz.name, COUNT(pv.id) AS count_of_visits FROM pizzeria pzz
	JOIN person_visits pv ON pv.pizzeria_id = pzz.id
	GROUP BY 1
	ORDER BY count_of_visits DESC
	LIMIT 3
), fav_rest_order AS (
	SELECT pzz.name, COUNT(pd.id) AS count_of_orders FROM pizzeria pzz
	JOIN menu ON menu.pizzeria_id = pzz.id
	JOIN person_order pd ON pd.menu_id = menu.id
	GROUP BY 1
	ORDER BY count_of_orders DESC
	LIMIT 3
)

SELECT fro.name, count_of_orders AS count, ('order') AS action_type FROM fav_rest_order fro
UNION
SELECT frv.name, count_of_visits AS count, ('visit') AS action_type FROM fav_rest_visit frv
ORDER BY 3 ASC, 2 DESC
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/d0df7615-4b8d-4895-b659-4afceb478f8c)

# 4
```
WITH fav_rest_visit AS (
	SELECT pzz.name, COUNT(pv.id) AS count_of_visits FROM pizzeria pzz
	JOIN person_visits pv ON pv.pizzeria_id = pzz.id
	GROUP BY 1
	ORDER BY count_of_visits DESC
), fav_rest_order AS (
	SELECT pzz.name, COUNT(pd.id) AS count_of_orders FROM pizzeria pzz
	JOIN menu ON menu.pizzeria_id = pzz.id
	JOIN person_order pd ON pd.menu_id = menu.id
	GROUP BY 1
	ORDER BY count_of_orders DESC
)

SELECT frv.name, (frv.count_of_visits + fro.count_of_orders) AS total_count FROM fav_rest_visit frv
JOIN fav_rest_order fro ON fro.name = frv.name
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/0f23571a-3ab3-4b4b-ba61-668582348909)

# 5
```
SELECT p.name, COUNT(pv.id) AS count_of_visits FROM person p
JOIN person_visits pv ON pv.person_id = p.id
GROUP BY 1
HAVING COUNT(pv.id) > 3
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/e6d08e44-d14f-4efc-96a7-609fa38320c6)

# 6
```
SELECT DISTINCT p.name FROM person p
LEFT JOIN person_order pd ON pd.person_id = p.id
ORDER BY 1
```
![image](https://github.com/VLSBorzykh/MrBeast1/assets/148325138/794c283d-fc2c-426c-b7e6-2ca183fee997)

# 7
```

