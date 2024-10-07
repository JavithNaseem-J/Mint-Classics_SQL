## Project Overview:
In this project, you will step into the shoes of an entry-level data analyst at the fictional Mint Classics Company, helping to analyze data in a relational database with the goal of supporting inventory-related business decisions that lead to the closure of a storage facility.


## Project Scenario:
Mint Classics Company, a retailer of classic model cars and other vehicles, is looking at closing one of their storage facilities. 
To support a data-based business decision, they are looking for suggestions and recommendations for reorganizing or reducing inventory, while still maintaining timely service to their customers



## Entity-Relationship Diagram

![image](https://github.com/user-attachments/assets/d2129a69-832c-4d8f-8102-e4106ccc0e2a)



## 1. How many unique products are currently in the inventory?
```sql
SELECT 
    COUNT(DISTINCT PRODUCTCODE) 
FROM 
    PRODUCTS;
```
![image](https://github.com/user-attachments/assets/fb7751c3-664f-478e-84fa-73d4f8d8a790)


## 2. List all the product categories and the count of products within each category
```sql
SELECT
    PRODUCTLINE AS PRODUT_CATEGORY, COUNT(*) AS NO_OF_PRODUCT
FROM
    PRODUCTS
GROUP BY
    1
ORDER BY
    2 DESC;
```
![image](https://github.com/user-attachments/assets/e10abe35-5130-4aa5-bfd9-1fa9210dd5fe)


## 3. What is the highest quantity and low quantity of each product available in the inventory?
```sql
WITH TOTALQUANTITYS AS (
	SELECT 
		PRODUCTNAME,SUM(QUANTITYINSTOCK) AS TOTAL_QUANTITY
	FROM 
		PRODUCTS
	GROUP BY 1)
SELECT  MAX(TOTAL_QUANTITY) AS HIGHEST_QUANTITY,MIN(TOTAL_QUANTITY) AS LOWEST_QUANTITY FROM TOTALQUANTITYS;
```
![image](https://github.com/user-attachments/assets/56ccbe11-54b7-4ca0-b4bb-7bca474c424e)


## 4. Current storage location of each product
```sql
SELECT 
    PRODUCTNAME AS PRODUCT_CATEGORY,WAREHOUSENAME AS WAREHOUSE_LOCATION,SUM(QUANTITYINSTOCK) AS TOTAL_QUANTITY
FROM 
    PRODUCTS P
INNER JOIN
    WAREHOUSES W
ON
    P.WAREHOUSECODE=W.WAREHOUSECODE
GROUP BY 
    1,2;
```
![sCtgCKdzRV](https://github.com/user-attachments/assets/6027453e-4367-4181-bdb3-5aafab38146a)



## 5. Unique product count and total stock in each warehouse
This query shows the number of unique products and the total quantity in each warehouse.
```sql
SELECT 
    P.WAREHOUSECODE,WAREHOUSENAME,COUNT(DISTINCT PRODUCTCODE),SUM(QUANTITYINSTOCK) AS TOTAL_QUANTITY 
FROM
    PRODUCTS AS P
INNER JOIN 
    WAREHOUSES AS W
ON
    P.WAREHOUSECODE=W.WAREHOUSECODE
GROUP BY
    1,2;
```
![image](https://github.com/user-attachments/assets/69b70ae1-bce4-45e4-88d9-d09600683e1b)


## 6. Products with low quantities that can be consolidated into fewer storage locations
This query helps identify products with low stock that could potentially be consolidated.
```sql
SELECT 
    P.WAREHOUSECODE,PRODUCTCODE,PRODUCTNAME,SUM(QUANTITYINSTOCK) AS TOTAL_QUANTITY
FROM
    PRODUCTS AS P
INNER JOIN 
    WAREHOUSES AS W ON P.WAREHOUSECODE=W.WAREHOUSECODE
GROUP BY
    1,2,3
ORDER BY 
    4 ASC;
```
![Wwp0KUaJVm](https://github.com/user-attachments/assets/5cb71083-e865-4c0e-b817-179ad30d7fa4)


## 7. Top 10 customers who bought the most products
This query identifies the top 10 customers based on the total quantity of products bought.
```sql
SELECT 
    C.CUSTOMERNUMBER,CUSTOMERNAME,SUM(OD.QUANTITYORDERED) AS TOTAL_SOLD
FROM 
    CUSTOMERS C
INNER JOIN 
    ORDERS O ON C.CUSTOMERNUMBER=O.CUSTOMERNUMBER
INNER JOIN 
    ORDERDETAILS OD ON O.ORDERNUMBER=OD.ORDERNUMBER
GROUP BY 
    1,2
ORDER BY 
    3 DESC
LIMIT 10;
```
![image](https://github.com/user-attachments/assets/4e1aa895-ebd6-478a-aed7-c333b78b080e)


## 8. Relation between inventory numbers and sales figures
This query evaluates if the current inventory levels are appropriate based on sales data.
```sql
WITH CTE AS (SELECT 
    P.productCode, P.productName,P.quantityInStock AS current_inventory,SUM(OD.quantityOrdered) AS TOTAL_SOLD,
   	CASE 
        WHEN SUM(OD.quantityOrdered) IS NULL THEN 'No sales data'
        WHEN P.quantityInStock > SUM(OD.quantityOrdered) THEN 'OVERSTOCK'
        WHEN P.quantityInStock < SUM(OD.quantityOrdered) THEN 'UNDERSTOCK'
        ELSE 'PPROPRIATE STOCK'
    END AS STOCK_STATUS
FROM 
    PRODUCTS P
LEFT JOIN 
    ORDERDETAILS OD ON P.PRODUCTCODE = OD.PRODUCTCODE
LEFT JOIN 
    ORDERS O ON OD.ORDERNUMBER = O.ORDERNUMBER
GROUP BY 
	1,2,3)
SELECT STOCK_STATUS,COUNT(STOCK_STATUS) FROM CTE
GROUP BY 1;
```
![image](https://github.com/user-attachments/assets/c0a1ac88-0ba2-41a1-a8bf-719a44bd790c)


## 9. Products with high inventory but low sales and optimizing inventory
This query identifies products that have high inventory levels but low sales, helping to optimize stock levels.
```sql
WITH INVENTORYSTORAGE AS (
    SELECT 
        P.PRODUCTCODE,PRODUCTNAME,QUANTITYINSTOCK,SUM(QUANTITYORDERED) AS TOTAL_SOLD
    FROM
        PRODUCTS P
    LEFT JOIN
        ORDERDETAILS O ON  P.PRODUCTCODE = O.PRODUCTCODE
    GROUP BY
        1,2,3)
SELECT 
    PRODUCTCODE,PRODUCTNAME,QUANTITYINSTOCK,TOTAL_SOLD,(QUANTITYINSTOCK-TOTAL_SOLD) AS INVENTORY FROM INVENTORYSTORAGE
WHERE 
    (QUANTITYINSTOCK-TOTAL_SOLD) > 0
ORDER BY 5 DESC;
```
![l1vltDYhqJ](https://github.com/user-attachments/assets/632b758c-be1f-4cc7-bedd-84d96389e2ab)


## 10. Relationship between product prices and their sales levels
This query examines if there's a correlation between product prices and sales volume.
```sql
SELECT 
    P.PRODUCTCODE,PRODUCTNAME,BUYPRICE,SUM(QUANTITYORDERED) AS TOTAL_SOLD
FROM 
    PRODUCTS P
LEFT JOIN 
    ORDERDETAILS OD ON P.PRODUCTCODE = OD.PRODUCTCODE
GROUP BY
    1,2,3
ORDER BY
    3 DESC;
```

## 11. Products with low quantities that can be consolidated
This query identifies products with less than 100 units in stock that may benefit from consolidation.
```sql
WITH LOW_STOCK AS (
    SELECT 
        PRODUCTCODE, SUM(quantityInStock) AS total_quantity
    FROM 
        public.products
    GROUP BY 
        PRODUCTCODE
    HAVING 
        SUM(quantityInStock) < 100
)
SELECT 
    P.PRODUCTCODE, PRODUCTNAME, WAREHOUSECODE, QUANTITYINSTOCK
FROM 
    PRODUCTS P
JOIN 
    LOW_STOCK LSP ON P.PRODUCTCODE = LSP.PRODUCTCODE
ORDER BY 
    1;
```
![image](https://github.com/user-attachments/assets/9ff06440-8da8-49ab-b62e-34e4559b9ff8)


## 12. Performance comparison of various product lines
This query compares the performance of different product lines based on inventory, sales, and revenue data.
```sql
SELECT 
    P.PRODUCTLINE,SUM(QUANTITYINSTOCK) AS TOTAL_INVENTORY,SUM(QUANTITYORDERED) AS TOTAL_SALES,SUM(PRICEEACH * QUANTITYORDERED) AS TOTAL_REVENUE,((SUM(QUANTITYORDERED)/SUM(QUANTITYINSTOCK))*100) AS SALESINVENTORYPERC
FROM
    PRODUCTS P
INNER JOIN
    PRODUCTLINES PL  ON P.PRODUCTLINE=PL.PRODUCTLINE
LEFT JOIN
    ORDERDETAILS OD ON P.PRODUCTCODE=OD.PRODUCTCODE
GROUP BY 
    1;
```
