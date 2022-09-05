# Sales Management Analysis

<img align="right" alt="Snoring Partner" width="1000" height = "400" src="https://i0.wp.com/leonine.com.ng/new/wp-content/uploads/2020/03/Leonine-Advisory-Page-Routine-Advisory-Services.jpg?resize=1024%2C729&ssl=1">

---


# Table of Contents

- [Introduction](https://github.com/globalsmile/Sales-Management-Analysis#introduction)
- [Problem Statement](https://github.com/globalsmile/Sales-Management-Analysis#Problem-Statement)
- [Data Sourcing](https://github.com/globalsmile/Sales-Management-Analysis#Data-Sourcing)
- [Data Preparation](https://github.com/globalsmile/Sales-Management-Analysis#Data-Preparation)
- [Data Modeling](https://github.com/globalsmile/Sales-Management-Analysis#Data-Modeling)
- [Data Visualization](https://github.com/globalsmile/Sales-Management-Analysis#Data-Visualization)
- [Data Analysis](https://github.com/globalsmile/Sales-Management-Analysis#Data-Analysis)
- [Insights](https://github.com/globalsmile/Sales-Management-Analysis#Insights)
- [Shareable link](https://github.com/globalsmile/Sales-Management-Analysis#Shareable-Link)


---

# Introduction

The goal of this sales management analysis is to create an executive sales report for sales managers.

Essentially, we want to focus it on how much we have sold of what products, to which clients and how it has been over time

---

# Problem Statement

Based on the request that was made from the business we are following the user stories that were defined to fulfill delivery and ensure that acceptance criteria’s were maintained throughout the project.

| #	As a (role) | I want (request / demand) | So that I (user value) | Acceptance Criteria |
| ----------- | ----------- | ----------- | ----------- |
| 1	Sales Manager | To get a dashboard overview of internet sales | Can follow better which customers and products sells the best | A Power BI dashboard which updates data once a day |
| 2	Sales Representative | A detailed overview of Internet Sales per Customers | Can follow up my customers that buys the most and who we can sell more to | A Power BI dashboard which allows me to filter data for each customer |
| 3	Sales Representative | A detailed overview of Internet Sales per Products | Can follow up my Products that sells the most | A Power BI dashboard which allows me to filter data for each Product |
| 4	Sales Manager | A dashboard overview of internet sales | Follow sales over time against budget | A Power Bi dashboard with graphs and KPIs comparing against budget. |


---

# Data Sourcing

- The Dataset used for this analysis was sourced from [AdventureWorks sample databases](https://docs.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms) 
- The additional dataset used for this analysis is available at: [Sales Budget](https://github.com/globalsmile/Sales-Management-Analysis/blob/main/SalesBudget.xlsx)
- The query used to update the database can be found at [Update Query](https://github.com/globalsmile/Sales-Management-Analysis/blob/main/Update_AdventureWorksDW_Data.sql)

---

# Data Preparation

Data cleaning and transformation was done in Microsoft SQL Server and the datasets was loaded into Microsoft Power BI Desktop for modeling.

The sales management datasets consists of 5 tables:

- `DIM_Calender` which has `8 columns and 1461 rows` of observation
- `DIM_Customers` which has `7 columns and 18484 rows` of observation
- `DIM_Products` which has `11 columns and 606 rows` of observation
- `FACT_InternetSales` which has `7 columns and 58168 rows` of observation
- `FACT_Budget` which has `2 columns and 18 rows` of observation

Below are the SQL statements for cleansing and transforming necessary data.

`DIM_Calender`

```TSQL
-- Cleansed DIM_Date Table --
SELECT 
  [DateKey], 
  [FullDateAlternateKey] AS Date, 
  --[DayNumberOfWeek], 
  [EnglishDayNameOfWeek] AS Day, 
  --[SpanishDayNameOfWeek], 
  --[FrenchDayNameOfWeek], 
  --[DayNumberOfMonth], 
  --[DayNumberOfYear], 
  --[WeekNumberOfYear],
  [EnglishMonthName] AS Month, 
  Left([EnglishMonthName], 3) AS MonthShort,   -- Useful for front end date navigation and front end graphs.
  --[SpanishMonthName], 
  --[FrenchMonthName], 
  [MonthNumberOfYear] AS MonthNo, 
  [CalendarQuarter] AS Quarter, 
  [CalendarYear] AS Year --[CalendarSemester], 
  --[FiscalQuarter], 
  --[FiscalYear], 
  --[FiscalSemester] 
FROM 
 [AdventureWorksDW2019].[dbo].[DimDate]
WHERE 
  CalendarYear >= 2019
```

`DIM_Customers`

```TSQL
-- Cleansed DIM_Customers Table --
SELECT 
  c.customerkey AS CustomerKey, 
  --      ,[GeographyKey]
  --      ,[CustomerAlternateKey]
  --      ,[Title]
  c.firstname AS [First Name], 
  --      ,[MiddleName]
  c.lastname AS [Last Name], 
  c.firstname + ' ' + lastname AS [Full Name], 
  -- Combined First and Last Name
  --      ,[NameStyle]
  --      ,[BirthDate]
  --      ,[MaritalStatus]
  --      ,[Suffix]
  CASE c.gender WHEN 'M' THEN 'Male' WHEN 'F' THEN 'Female' END AS Gender,
  --      ,[EmailAddress]
  --      ,[YearlyIncome]
  --      ,[TotalChildren]
  --      ,[NumberChildrenAtHome]
  --      ,[EnglishEducation]
  --      ,[SpanishEducation]
  --      ,[FrenchEducation]
  --      ,[EnglishOccupation]
  --      ,[SpanishOccupation]
  --      ,[FrenchOccupation]
  --      ,[HouseOwnerFlag]
  --      ,[NumberCarsOwned]
  --      ,[AddressLine1]
  --      ,[AddressLine2]
  --      ,[Phone]
  c.datefirstpurchase AS DateFirstPurchase, 
  --      ,[CommuteDistance]
  g.city AS [Customer City] -- Joined in Customer City from Geography Table
FROM 
  [AdventureWorksDW2019].[dbo].[DimCustomer] as c
  LEFT JOIN dbo.dimgeography AS g ON g.geographykey = c.geographykey 
ORDER BY 
  CustomerKey ASC -- Ordered List by CustomerKey
```

`DIM_Products`

```TSQL
-- Cleansed DIM_Products Table --
SELECT 
  p.[ProductKey], 
  p.[ProductAlternateKey] AS ProductItemCode, 
  --      ,[ProductSubcategoryKey], 
  --      ,[WeightUnitMeasureCode]
  --      ,[SizeUnitMeasureCode] 
  p.[EnglishProductName] AS [Product Name], 
  ps.EnglishProductSubcategoryName AS [Sub Category], -- Joined in from Sub Category Table
  pc.EnglishProductCategoryName AS [Product Category], -- Joined in from Category Table
  --      ,[SpanishProductName]
  --      ,[FrenchProductName]
  --      ,[StandardCost]
  --      ,[FinishedGoodsFlag] 
  p.[Color] AS [Product Color], 
  --      ,[SafetyStockLevel]
  --      ,[ReorderPoint]
  --      ,[ListPrice] 
  p.[Size] AS [Product Size], 
  --      ,[SizeRange]
  --      ,[Weight]
  --      ,[DaysToManufacture]
  p.[ProductLine] AS [Product Line], 
  --     ,[DealerPrice]
  --      ,[Class]
  --      ,[Style] 
  p.[ModelName] AS [Product Model Name], 
  --      ,[LargePhoto]
  p.[EnglishDescription] AS [Product Description], 
  --      ,[FrenchDescription]
  --      ,[ChineseDescription]
  --      ,[ArabicDescription]
  --      ,[HebrewDescription]
  --      ,[ThaiDescription]
  --      ,[GermanDescription]
  --      ,[JapaneseDescription]
  --      ,[TurkishDescription]
  --      ,[StartDate], 
  --      ,[EndDate], 
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
-- Cleansed FACT_InternetSales Table --
SELECT 
  [ProductKey], 
  [OrderDateKey], 
  [DueDateKey], 
  [ShipDateKey], 
  [CustomerKey], 
  --  ,[PromotionKey]
  --  ,[CurrencyKey]
  --  ,[SalesTerritoryKey]
  [SalesOrderNumber], 
  --  [SalesOrderLineNumber], 
  --  ,[RevisionNumber]
  --  ,[OrderQuantity], 
  --  ,[UnitPrice], 
  --  ,[ExtendedAmount]
  --  ,[UnitPriceDiscountPct]
  --  ,[DiscountAmount] 
  --  ,[ProductStandardCost]
  --  ,[TotalProductCost] 
  [SalesAmount] --  ,[TaxAmt]
  --  ,[Freight]
  --  ,[CarrierTrackingNumber] 
  --  ,[CustomerPONumber] 
  --  ,[OrderDate] 
  --  ,[DueDate] 
  --  ,[ShipDate] 
FROM 
  [AdventureWorksDW2019].[dbo].[FactInternetSales]
WHERE 
  LEFT (OrderDateKey, 4) >= YEAR(GETDATE()) -2 -- Ensures we always only bring two years of date from extraction.
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

# Data Visualization

Data visualization for the datasets was done in 3 folds using Microsoft Power BI Desktop:

- The `Sales Overview`: Shows the sales vs budget KPI, sales by top 10 customers, sales by top 10 products, e.t.c
-  The `Customer Details`: Shows customer specific information
-  The `Product Details`: Shows product specific information


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

---

# Data Analysis & Insights

Measures used in visualization are:

- Budget = `SUM ( FACT_Budget[Budget] )`
- Sales = `SUM ( FACT_InternetSales[SalesAmount] )`
- Sales / Budget = `[Sales] - [Budget]`


As shown from [Data Visualization](https://github.com/globalsmile/Sales-Management-Analysis#Data-Visualization), It can be deducted that for the year ending December 2021:

- Sales is up by `1,051,550`
- The top customer by sales is `Jordan Turner`
- The top product by sales is `Mountain-200 Black,42`
- The top product category by sales is the `Bikes` occupying 93.93% of the total sales
- There is a positive correlation between `Sales` and `Budget`

---

# Shareable Link

You can interact with the report here: 

[Dashboard](https://app.powerbi.com/view?r=eyJrIjoiNWVmNDRjNjEtNjZjZC00OGE2LTllYjktMDRjMmMyOTE4ZDYxIiwidCI6IjQ5ODY4YWYzLWNjNWYtNDIxNC04YjdmLTQwZjM3NDY0OWEwOSJ9)
