#Задание 1

SELECT Customers.customer_id, Customers.first_name, Customers.last_name FROM Customers WHERE ( SELECT COUNT(Orders.order_id) FROM Orders WHERE Orders.customer_id = Customers.customer_id AND Orders.order_date >= current_date - interval '3 months' ) > 2;

image

#Задание 2

SELECT Products.category, AVG(Orders.quantity) AS average_order_size FROM Products JOIN Orders ON Products.product_id = Orders.product_id WHERE Products.price >= 50 GROUP BY Products.category;

image

#Задание 3

WITH AverageOrderCost AS ( SELECT AVG(p.price * o.quantity) AS avg_order_cost FROM Orders o JOIN Products p ON o.product_id = p.product_id ) SELECT c.customer_id, c.first_name, c.last_name, SUM(p.price * o.quantity) AS total_order_cost FROM Customers c JOIN Orders o ON c.customer_id = o.customer_id JOIN Products p ON o.product_id = p.product_id GROUP BY c.customer_id, c.first_name, c.last_name HAVING SUM(p.price * o.quantity) > (SELECT avg_order_cost FROM AverageOrderCost) ORDER BY total_order_cost DESC;

image

#Задание 4

SELECT c.customer_id, c.first_name, c.last_name FROM Customers c WHERE c.customer_id IN ( SELECT o.customer_id FROM Orders o JOIN Products p ON o.product_id = p.product_id WHERE p.category <> 'Electronics' GROUP BY o.customer_id HAVING SUM(p.price * o.quantity) > 1000 );

image

#Задание 5

CREATE OR REPLACE VIEW CustomerOrderStats AS SELECT Customers.customer_id, Customers.first_name, Customers.last_name, AVG(Products.price * Orders.quantity) AS avg_order_cost, AVG(Products.price * Orders.quantity) - ( SELECT AVG(Products.price * Orders.quantity) FROM Orders JOIN Products ON Orders.product_id = Products.product_id ) AS deviation FROM Customers LEFT JOIN Orders ON Customers.customer_id = Orders.customer_id LEFT JOIN Products ON Orders.product_id = Products.product_id GROUP BY Customers.customer_id, Customers.first_name, Customers.last_name;

#Задание 6

SELECT C.customer_id, C.first_name, C.last_name, MAX(date_diff) AS max_diff FROM Customers C JOIN ( SELECT O1.customer_id, MAX(O1.order_date - O2.order_date) AS date_diff FROM Orders O1 JOIN Orders O2 ON O1.customer_id = O2.customer_id WHERE O1.order_date > O2.order_date GROUP BY O1.customer_id ) AS max_diffs ON C.customer_id = max_diffs.customer_id GROUP BY C.customer_id, C.first_name, C.last_name ORDER BY max_diff DESC LIMIT 1;

image

#Задание 7

WITH RecentOrderTotals AS ( SELECT o.customer_id, SUM(p.price * o.quantity) AS total_cost FROM Orders o JOIN Products p ON o.product_id = p.product_id WHERE o.order_date >= (CURRENT_DATE - INTERVAL '2 months') GROUP BY o.customer_id ), PreviousOrderTotals AS ( SELECT o.customer_id, SUM(p.price * o.quantity) AS total_cost FROM Orders o JOIN Products p ON o.product_id = p.product_id WHERE o.order_date >= (CURRENT_DATE - INTERVAL '4 months') AND o.order_date < (CURRENT_DATE - INTERVAL '2 months') GROUP BY o.customer_id ) SELECT c.customer_id, c.first_name, c.last_name FROM Customers c JOIN RecentOrderTotals recent ON c.customer_id = recent.customer_id LEFT JOIN PreviousOrderTotals previous ON c.customer_id = previous.customer_id WHERE recent.total_cost > COALESCE(previous.total_cost, 0);

image

#Задание 8

UPDATE Products SET price = price * 10 WHERE category = 'Clothing';

image

#Задание 9

WITH CategoryAvgPrice AS ( SELECT category, AVG(price) AS avg_price FROM Products GROUP BY category ) SELECT category, avg_price FROM CategoryAvgPrice WHERE avg_price = (SELECT MAX(avg_price) FROM CategoryAvgPrice) OR avg_price = (SELECT MIN(avg_price) FROM CategoryAvgPrice);

image

GROUP BY cus.first_name, cus.last_name

HAVING (SELECT MAX(rgxn.range) FROM range_max_min rgxn)
