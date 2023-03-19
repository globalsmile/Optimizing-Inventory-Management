# Optimizing Inventory Management

<img align="right" alt="Optimizing Inventory Management" width="1000" height = "500" src="https://user-images.githubusercontent.com/106287208/188423026-e1b4d76d-ffcc-4f8c-a1b7-bd363922f524.png">

---


# Table of Contents

- [Introduction](https://github.com/globalsmile/Optimizing-Inventory-Management#Introduction)
- [Problem Statement](https://github.com/globalsmile/Optimizing-Inventory-Management#Problem-Statement)
- [Data Sourcing](https://github.com/globalsmile/Optimizing-Inventory-Management#Data-Sourcing)
- [Data Preparation](https://github.com/globalsmile/Optimizing-Inventory-Management#Data-Preparation)
- [Data Modeling](https://github.com/globalsmile/Optimizing-Inventory-Management#Data-Modeling)
- [Data Visualization and Analysis](https://github.com/globalsmile/Optimizing-Inventory-Management#Data-Visualization-and-Analysis)
- [Shareable link](https://github.com/globalsmile/Optimizing-Inventory-Management#Shareable-Link)


---

# Introduction

SellCheapy Retail is a chain of department stores that sells a wide range of products, including bikes and different components. Despite having a large customer base, the company has been struggling to 
increase sales in recent years. The management team is looking to use data analysis to understand customer spending patterns and make changes to their sales and marketing strategies to improve performance.

---

# Problem Statement

Based on the request that was made from the management team we are following the user stories that were defined to fulfill delivery and ensure that acceptance criteria’s were maintained throughout the project.

| #	As a (role) | I want (request / demand) | So that I (user value) | Acceptance Criteria |
| ----------- | ----------- | ----------- | ----------- |
| 1	Sales Manager | To get a dashboard overview of internet sales | Can follow better which customers and products sells the best | A Power BI dashboard which updates data once a day |
| 2	Sales Representative | A detailed overview of Internet Sales per Customers | Can follow up my customers that buys the most and who we can sell more to | A Power BI dashboard which allows me to filter data for each customer |
| 3	Sales Representative | A detailed overview of Internet Sales per Products | Can follow up my Products that sells the most | A Power BI dashboard which allows me to filter data for each Product |
| 4	Sales Manager | A dashboard overview of internet sales | Follow sales over time against budget | A Power BI dashboard with graphs and KPIs comparing against budget. |


---

# Data Sourcing

- The data set used for this analysis was sourced from [AdventureWorks sample databases](https://docs.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms) 
- The additional data set used for this analysis is available at: [Sales Budget](https://github.com/globalsmile/Optimizing-Inventory-Management/blob/main/SalesBudget.xlsx)
- The query used to update the database can be found at [Update Query](https://github.com/globalsmile/Sales-Management-Analysis/blob/main/Update_AdventureWorksDW_Data.sql)

---

# Data Preparation

Data cleaning and transformation was done in Microsoft SQL Server and the data sets was loaded into Microsoft Power BI Desktop for modeling.

The data sets are split into 5 tables:

- `DIM_Calender` contains `8 columns and 1461 rows` 
- `DIM_Customers` contains `7 columns and 18484 rows`
- `DIM_Products` contains `11 columns and 606 rows`
- `FACT_InternetSales` contains `7 columns and 58168 rows`
- `FACT_Budget` which has `2 columns and 18 rows` of observation

Below are the SQL statements for cleansing and transforming necessary data.

`DIM_Calender`

```TSQL
SELECT 
  [DateKey], 
  [FullDateAlternateKey] AS Date, 
  [EnglishDayNameOfWeek] AS Day, 
  [EnglishMonthName] AS Month, 
  Left([EnglishMonthName], 3) AS MonthShort,  
  [MonthNumberOfYear] AS MonthNo, 
  [CalendarQuarter] AS Quarter, 
  [CalendarYear] AS Year --[CalendarSemester], 
FROM 
 [AdventureWorksDW2019].[dbo].[DimDate]
WHERE 
  CalendarYear >= 2019
```

`DIM_Customers`

```TSQL
SELECT 
  c.customerkey AS CustomerKey, 
  c.firstname AS [First Name], 
  c.lastname AS [Last Name], 
  c.firstname + ' ' + lastname AS [Full Name], 
  CASE c.gender WHEN 'M' THEN 'Male' WHEN 'F' THEN 'Female' END AS Gender,
  c.datefirstpurchase AS DateFirstPurchase, 
  g.city AS [Customer City]
FROM 
  [AdventureWorksDW2019].[dbo].[DimCustomer] as c
  LEFT JOIN dbo.dimgeography AS g ON g.geographykey = c.geographykey 
ORDER BY 
  CustomerKey ASC
```

`DIM_Products`

```TSQL
SELECT 
  p.[ProductKey], 
  p.[ProductAlternateKey] AS ProductItemCode, 
  p.[EnglishProductName] AS [Product Name], 
  ps.EnglishProductSubcategoryName AS [Sub Category], 
  pc.EnglishProductCategoryName AS [Product Category], 
  p.[Color] AS [Product Color], 
  p.[Size] AS [Product Size], 
  p.[ProductLine] AS [Product Line], 
  p.[ModelName] AS [Product Model Name], 
  p.[EnglishDescription] AS [Product Description], 
  ISNULL (p.Status, 'Outdated') AS [Product Status] 
FROM 
  [AdventureWorksDW2019].[dbo].[DimProduct] as p
  LEFT JOIN dbo.DimProductSubcategory AS ps ON ps.ProductSubcategoryKey = p.ProductSubcategoryKey 
  LEFT JOIN dbo.DimProductCategory AS pc ON ps.ProductCategoryKey = pc.ProductCategoryKey 
order by 
  p.ProductKey asc
```

`FACT_InternetSales`

```TSQL
SELECT 
  [ProductKey], 
  [OrderDateKey], 
  [DueDateKey], 
  [ShipDateKey], 
  [CustomerKey], 
  [SalesOrderNumber], 
  [SalesAmount] 
FROM 
  [AdventureWorksDW2019].[dbo].[FactInternetSales]
WHERE 
  LEFT (OrderDateKey, 4) >= YEAR(GETDATE()) -2
ORDER BY
  OrderDateKey ASC
```

The tabulation below shows the `FACT_Budget` table with its column names and their description:
| Column Name | Description |
| ----------- | ----------- |
| Date | Represents the date budget was made|
| Budget | Represents the amount budgeted |

---

# Data Modeling

After the dataset was cleaned and transformed, it was ready to be modeled(using Power BI Desktop).

- The `DIM_Calender` table was marked as the official date table in the dataset.
- A `one-to-many (*:1) relationship` was created between the `Fact_InternetSales` and the `DIM_Calender` tables 
- A `one-to-many (*:1) relationship` was created between the `Fact_InternetSales` and the `DIM_Customers` tables 
- A `one-to-many (*:1) relationship` was created between the `Fact_InternetSales` and the `DIM_Products` tables 
- A `one-to-many (*:1) relationship` was created between the `FACT_Budget` and the `DIM_Calender` tables using 
- The realtioships formed is shown in the data model below:

<img align="right" alt="Data Model" width="1000" height = "400" src="https://user-images.githubusercontent.com/106287208/188418211-26dbee49-71c9-4fef-b767-279d6d4d67cc.png">


---

# Data Visualization and Analyis

Data visualization for the datasets was done in 3 folds using Microsoft Power BI Desktop.

Based on the business problem, the following report was prepared:

- The `Sales Overview` page: contains the sales vs budget KPI, sales by top 10 customers, sales by top 10 products, e.t.c
-  The `Customer Details` page: contains customer specific information
-  The `Product Details` page: contains product specific information


Figure 1 shows visualizations from `Sales Overview` page

| Figure 1 |
| ----------- |
| ![image](https://user-images.githubusercontent.com/106287208/188423026-e1b4d76d-ffcc-4f8c-a1b7-bd363922f524.png) |

Figure 2 shows visualizations from `Customer Details` page

| Figure 2 |
| ----------- |
| ![image](https://user-images.githubusercontent.com/106287208/188423353-266e6a75-d31a-4f2e-ba1d-30dd1f591050.png) |

Figure 3 shows visualizations from `Product Details` page

| Figure 3 |
| ----------- |
| ![image](https://user-images.githubusercontent.com/106287208/188423526-fe1c6dd3-73da-4f8f-b757-5d585be1eec3.png) |


Measures used in this visualizations are:

- Budget = `SUM ( FACT_Budget[Budget] )`
- Sales = `SUM ( FACT_InternetSales[SalesAmount] )`
- Sales / Budget = `[Sales] - [Budget]`


---


# Shareable Link

You can interact with the report here: 

[View Report](https://app.powerbi.com/view?r=eyJrIjoiNWVmNDRjNjEtNjZjZC00OGE2LTllYjktMDRjMmMyOTE4ZDYxIiwidCI6IjQ5ODY4YWYzLWNjNWYtNDIxNC04YjdmLTQwZjM3NDY0OWEwOSJ9)
