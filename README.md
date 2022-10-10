# Dataquest.Project1 (Guided Project)

```
SELECT *
  FROM products
 LIMIT 5;
 ```
 ```
SELECT COUNT(*)
  FROM products;
```
-- QUESRY to display the following table
```
SELECT "customers",
count(*) as num_of_rows,
13 AS number_of_attribute
from customers
UNION
SELECT "products",
count(*) as num_of_rows,
9 AS number_of_attribute
from products
UNION
SELECT "productlines",
count(*) as num_of_rows,
4 AS number_of_attribute
from productlines
UNION
SELECT "orders",
count(*) as num_of_rows,
7 AS number_of_attribute
from orders
UNION
SELECT "orderdetails",
count(*) as num_of_rows,
5 AS number_of_attribute
from orderdetails
UNION
SELECT "payments",
count(*) as num_of_rows,
4 AS number_of_attribute
from payments
UNION
SELECT "employees",
count(*) as num_of_rows,
8 AS number_of_attribute
from employees
UNION
SELECT "offices",
count(*) as num_of_rows,
9 AS number_of_attribute
from offices;
```
--2. product performance
```
SELECT productCode, Sum(quantityOrdered * priceEach) as prod_perf
from orderdetails
group by productCode
order by prod_perf DESC
limit 10;
```
-- 3. low stock

```
select od.productCode,
Round(Sum(quantityOrdered) * 1.0/(select quantityInStock from products p where p.productCode = od.productCode),2) as low_stock
from orderdetails od
GROUP by od.productCode
order by low_stock ASC
limit 10;
```
-- 4.Priority Products for restocking
```
WITH 
low_stock_table AS (
SELECT productCode,
       ROUND(SUM(quantityOrdered) * 1.0/(SELECT quantityInStock
                                           FROM products p
                                          WHERE od.productCode = p.productCode), 2) AS low_stock
  FROM orderdetails od
 GROUP BY productCode
 ORDER BY low_stock
 LIMIT 10
)
SELECT productCode,
       SUM(quantityOrdered * priceEach) AS prod_perf
  FROM orderdetails od
 WHERE productCode IN (SELECT productCode
                         FROM low_stock_table)
 GROUP BY productCode 
 ORDER BY prod_perf DESC
 LIMIT 10;
 ```
 -- Name of products for restocking
```
WITH
  perform AS (
    SELECT productCode,
           SUM(quantityOrdered) * 1.0 AS qntOrdr,	
           SUM(quantityOrdered * priceEach) AS prod_perf
      FROM orderdetails
     GROUP BY productCode
  ),
  low_stock_table AS (
    SELECT p.productCode, 
	       p.productName, 
		   p.productLine,
           ROUND(SUM(perform.qntOrdr * 1.0) / p.quantityInstock, 2) AS low_stock
      FROM products p
	  JOIN perform
	    ON p.productCode = perform.productCode
     GROUP BY p.productCode
	 ORDER BY low_stock
	 LIMIT 10
  )
    SELECT low_stock_table.productName, 
	       low_stock_table.productLine
	  FROM low_stock_table
	  JOIN perform
	    ON low_stock_table.productCode = perform.productCode
	 ORDER BY perform.prod_perf DESC;
 ```
 -- 5.revenue by customer
```
SELECT o.customerNumber,p.productLine,p.productName, SUM(quantityOrdered * (priceEach - buyPrice)) AS revenue
  FROM products p
  JOIN orderdetails od
    ON p.productCode = od.productCode
  JOIN orders o
    ON o.orderNumber = od.orderNumber
 GROUP BY o.customerNumber,p.productLine,p.productName;
 
--VIP Customers
with
VIP_customer AS 
(
SELECT o.customerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS profit
FROM products p
JOIN orderdetails od
ON p.productCode = od.productCode
JOIN orders o
ON o.orderNumber = od.orderNumber
GROUP BY o.customerNumber
)
SELECT contactLastName, contactFirstName, city, country, VIP.profit
FROM customers c
JOIN VIP_customer VIP
ON VIP.customerNumber = c.customerNumber
ORDER BY VIP.profit DESC
LIMIT 5;
```
-- LEAST ENGAGED customers
```
with
Least_engaged_customer AS 
(
SELECT o.customerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS profit
FROM products p
JOIN orderdetails od
ON p.productCode = od.productCode
JOIN orders o
ON o.orderNumber = od.orderNumber
GROUP BY o.customerNumber
)
SELECT contactLastName, contactFirstName, city, country, LE.profit
FROM customers c
JOIN Least_engaged_customer LE
ON LE.customerNumber = c.customerNumber
ORDER BY LE.profit ASC
LIMIT 5;
```
-- LTV (Sub query)


```
SELECT AVG(mc.profit) as avg_money
from (SELECT o.customerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS profit
        FROM products p
        JOIN orderdetails od
          ON p.productCode = od.productCode
        JOIN orders o
          ON o.orderNumber = od.orderNumber
    GROUP BY o.customerNumber) AS mc
```
