
## 1. How many unique products are currently in the inventory?
This query retrieves the total number of unique products available in the inventory.
```sql
SELECT 
    COUNT(DISTINCT PRODUCTCODE) 
FROM 
    PRODUCTS;
```

## 2. List all the product categories and the count of products within each category
This query lists the product categories and the number of products within each.
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

## 3. Total quantity of each product available in the inventory
This query provides the total stock quantity for each product.
```sql
SELECT 
    PRODUCTNAME,SUM(QUANTITYINSTOCK) AS TOTAL_QUANTITY
FROM 
    PRODUCTS
GROUP BY
    1
ORDER BY
    2 DESC;
```

## 4. Current storage location of each product
This query shows the warehouse location for each product, along with the total quantity in stock.
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

## 5. Verify if any products are stored in multiple warehouses
This query identifies products that are stored in more than one warehouse.
```sql
SELECT 
    PRODUCTCODE,COUNT(WAREHOUSECODE) AS WAREHOUSE
FROM 
    PRODUCTS
GROUP BY 
    PRODUCTCODE
HAVING 
    COUNT(WAREHOUSECODE) > 1;
```

## 6. Unique product count and total stock in each warehouse
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

## 7. Products with low quantities that can be consolidated into fewer storage locations
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

## 8. Top 10 customers who bought the most products
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

## 9. Relation between inventory numbers and sales figures
This query evaluates if the current inventory levels are appropriate based on sales data.
```sql
SELECT 
    P.productCode, P.productName,P.quantityInStock AS current_inventory,SUM(OD.quantityOrdered) AS TOTAL_SOLD,
    CASE 
        WHEN SUM(OD.quantityOrdered) IS NULL THEN 'No sales data'
        WHEN P.quantityInStock > SUM(OD.quantityOrdered) THEN 'OVERSTOCK'
        WHEN P.quantityInStock < SUM(OD.quantityOrdered) THEN 'UNDERSTOCK'
        ELSE 'APPROPRIATE STOCK'
    END AS STOCK_STATUS
FROM 
    PRODUCTS P
LEFT JOIN 
    ORDERDETAILS OD ON P.PRODUCTCODE = OD.PRODUCTCODE
LEFT JOIN 
    ORDERS O ON OD.ORDERNAME = O.ORDERNAME
GROUP BY 
    1,2,3;
```

## 10. Products with high inventory but low sales and optimizing inventory
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

## 11. Relationship between product prices and their sales levels
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

## 12. Products with low quantities that can be consolidated
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

## 13. Performance comparison of various product lines
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
