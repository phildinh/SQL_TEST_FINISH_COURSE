# Sales Data Analysis Project

## Overview
This project involves analyzing sales data to extract insights and make data-driven decisions. It starts with the data extraction from Excel, converting it into a more manageable CSV format, and finally loading it into an SQL database for complex querying.

## Introduction
The goal of this project is to demonstrate the ability to manipulate and process data through various stages—from raw data extraction, transformation, and loading (ETL) to executing complex SQL queries to analyze sales performance, employee productivity, and the impact of discounts on sales across various product categories.

## Data Preparation

### Step 1: Extracting Data from Excel
Data initially stored in Excel spreadsheets is extracted and prepared for transformation. This involves selecting relevant columns and ensuring data integrity.

### Step 2: Transforming to CSV
Once the Excel data is curated, it is converted into a CSV format. This transformation simplifies the loading process into the SQL database and helps in handling larger datasets more efficiently.

### Step 3: Loading Data into SQL
The CSV data is then loaded into an SQL database. This stage sets the foundation for performing detailed and complex queries to derive meaningful insights from the data.

## SQL Queries for Analysis
Following the data preparation, several SQL queries are executed to analyze the data from different perspectives. Below are the queries and their descriptions:

### Question 1: 
--Write a SQL query to calculate the total sales of furniture products, grouped by each quarter of the year, and order the results chronologically?
```sql
SELECT
	CONCAT('Q', DATEPART(QUARTER, o.ORDER_DATE), '-', YEAR(o.ORDER_DATE)) AS Quarter_Year,
	CAST(SUM(o.SALES) AS DECIMAL(10,2)) AS Total_Sales
FROM order1 AS o
JOIN product AS p ON o.PRODUCT_ID = p.ID
WHERE p.NAME = 'Furniture'
GROUP BY
	YEAR(o.ORDER_DATE),
	DATEPART(QUARTER, o.ORDER_DATE)
ORDER BY
	YEAR(o.ORDER_DATE),
	DATEPART(QUARTER, o.ORDER_DATE);
```

### Question 2:
--Write a query to analyze the impact of different discount levels on sales performance across product categories, 
--specifically looking at the number of orders and total profit generated for each discount classification?
--Discount level condition:
--No Discount = 0
--0 < Low Discount < 0.2
--0.2 < Medium Discount < 0.5
--High Discount > 0.5  
```sql
SELECT 
  p.CATEGORY, 
  CASE 
    WHEN o.DISCOUNT = 0 THEN 'No Discount'
    WHEN o.DISCOUNT > 0 AND o.DISCOUNT < 0.2 THEN 'Low Discount'
    WHEN o.DISCOUNT >= 0.2 AND o.DISCOUNT < 0.5 THEN 'Medium Discount'
    ELSE 'High Discount'
  END AS Discount_level,
  COUNT(o.ROW_ID) AS Total_orders, 
  CAST(SUM(o.PROFIT) AS DECIMAL(10,2)) AS Total_Profit
FROM order1 AS o JOIN product AS p 
ON o.PRODUCT_ID = p.ID
GROUP BY  p.CATEGORY, 
  CASE 
    WHEN o.DISCOUNT = 0 THEN 'No Discount'
    WHEN o.DISCOUNT > 0 AND o.DISCOUNT < 0.2 THEN 'Low Discount'
    WHEN o.DISCOUNT >= 0.2 AND o.DISCOUNT < 0.5 THEN 'Medium Discount'
    ELSE 'High Discount'
  END
ORDER BY p.CATEGORY ASC;
```
### Question 3:
----Write a query to determine the top-performing product categories within each customer segment based on sales and profit,
-- focusing specifically on those categories that rank within the top two for profitability?
```sql
WITH T1 AS (
SELECT 
	c.SEGMENT, 
	p.CATEGORY,
	SUM(o.SALES) AS Total_Sale, 
	SUM(o.PROFIT) AS Total_profit,
DENSE_RANK() OVER (PARTITION BY c.SEGMENT ORDER BY SUM(o.SALES) DESC) AS Sales_Rank,
DENSE_RANK() OVER (PARTITION BY c.SEGMENT ORDER BY SUM(o.PROFIT) DESC) AS Profit_Rank
FROM order1 AS o 
JOIN product as p ON o.PRODUCT_ID = p.ID
JOIN customer as c ON o.CUSTOMER_ID = c.ID
GROUP BY c.SEGMENT,p.CATEGORY)

SELECT T1.SEGMENT, T1.CATEGORY, T1.Sales_Rank, T1.Profit_Rank
FROM T1
WHERE T1.Profit_rank IN (1,2);
```
### Question 4:
--Write a query to create a report that displays each employee's performance across different product categories, 
--showing not only the total profit per category but also what percentage of their total profit each category represents, 
--with the results ordered by the percentage in descending order for each employee?
```sql
SELECT e.ID_EMPLOYEE, p.CATEGORY,
	CAST(SUM(o.PROFIT) AS DECIMAL(10,2)) AS Rounded_Total_Profit,
	CAST((SUM(o.PROFIT) / SUM(SUM(o.PROFIT)) OVER (PARTITION BY p.CATEGORY)) * 100 AS DECIMAL(10,1)) AS Profit_Percentage
FROM order1 AS o 
	JOIN product AS p	ON o.PRODUCT_ID = p.ID
	JOIN employee AS e	ON o.ID_EMPLOYEE= e.ID_EMPLOYEE
GROUP BY 
	e.ID_EMPLOYEE, p.CATEGORY
ORDER BY
	e.ID_EMPLOYEE ASC, Profit_Percentage DESC;
```
### Question 5: 
--Write a query to develop a user-defined function in SQL Server to calculate the profitability ratio for each product category an employee has sold, 
--and then apply this function to generate a report that ranks each employee's product categories by their profitability ratio?
```sql
WITH T1 AS (
SELECT e.ID_EMPLOYEE, p.CATEGORY,
	CAST(SUM(o.SALES) AS DECIMAL(10,2)) AS Total_Sales,
	CAST(SUM(o.PROFIT) AS DECIMAL(10,2)) AS Total_Profit,
	CAST((SUM(o.PROFIT) / (SUM(o.SALES)) * 100) AS DECIMAL(10,2)) AS Profitability_Ratio_Percent
FROM order1 AS o 
	JOIN product AS p	ON o.PRODUCT_ID = p.ID
	JOIN employee AS e	ON o.ID_EMPLOYEE= e.ID_EMPLOYEE
GROUP BY 
	e.ID_EMPLOYEE, p.CATEGORY
)

SELECT *,
	DENSE_RANK() OVER (PARTITION BY T1.CATEGORY ORDER BY Profitability_Ratio_Percent DESC) AS Rank_Employee_Category
FROM T1;
```
## Contact Information

For further information, collaboration, or inquiries, feel free to contact [Phil Dinh](mailto:dinhthanhtrung2011@gmail.com).
