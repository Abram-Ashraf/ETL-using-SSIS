# 📊 AdventureWorks ETL Data Warehouse Project  
**SQL Server Integration Services (SSIS) – Full Data Warehouse Pipeline**

---

## 📌 Project Overview

This project demonstrates a complete **Enterprise Data Warehouse (DWH) solution** built using:

- Microsoft SQL Server  
- SQL Server Integration Services (SSIS)  
- AdventureWorks2012 database  
- Layered architecture (ODS → STG → DWH)  

The goal of this project is to extract data from the AdventureWorks2012 OLTP system, transform it using SSIS, and load it into a structured **Star Schema Data Warehouse** optimized for analytics and reporting.

---

## 🗂 Source Database

### AdventureWorks2012

AdventureWorks2012 is Microsoft’s official sample OLTP database that simulates a manufacturing company selling bicycles and accessories.

It is used as the source system for this project.

### Key Source Tables Used

- Person.Person  
- Person.EmailAddress  
- Person.PersonPhone  
- HumanResources.Employee  
- Sales.SalesOrderHeader  
- Sales.SalesOrderDetail  
- Production.Product  
- Production.ProductSubcategory  
- Production.ProductCategory  
- Sales.Customer  
- Sales.SalesTerritory  
- Person.Address  

---

# 🏗 Architecture Overview


AdventureWorks2012 (OLTP)
│
▶
ODS Database
│
▶
STG Database
│
▶
DWH Database (Star Schema)


---

# 🧱 1️⃣ ODS Layer (Operational Data Store)

## 📌 Purpose

The ODS layer acts as a **clean, integrated snapshot of the source system**.

It:

- Extracts data from AdventureWorks
- Performs light cleaning
- Preserves original business keys
- Serves as the controlled source for staging

## 🗄 Database Name

`AdventureWorks_ODS`

## ⚙️ Processes in ODS

### 1. Full Extraction (Initial Load)

- Data extracted using SSIS Data Flow Tasks
- Source: OLE DB Connection to AdventureWorks2012
- Destination: ODS tables
- Full load strategy (truncate before load)

### 2. Light Transformations

- Standardized column names
- Converted NULL values
- Basic data type conversions
- Removed unnecessary technical columns

### 3. Tables Created in ODS

- ODS_Person
- ODS_Employee
- ODS_Product
- ODS_Customer
- ODS_SalesOrderHeader
- ODS_SalesOrderDetail

---

# 🧪 2️⃣ STG Layer (Staging Layer)

## 📌 Purpose

The STG layer prepares data for dimensional modeling.

It:

- Joins related tables
- Creates derived columns
- Applies business logic
- Prepares dimension-ready tables

## 🗄 Database Name

`AdventureWorks_STG`

---

## 🧠 Transformations in STG

### 🔹 Person Dimension Preparation

Sources:

- Person.Person  
- EmailAddress  
- PersonPhone  
- Address  

Created table:

`person_dim`

### Derived Columns Examples

- FullName = FirstName + ' ' + LastName
- Age = DATEDIFF(YEAR, BirthDate, GETDATE())
- Gender Description (M/F → Male/Female)
- Marital Status Description
- IsCurrentEmployee Flag
- EmailAvailable Flag

---

### 🔹 Product Dimension Preparation

Joined:

- Product
- ProductSubcategory
- ProductCategory

Created table:

`product_dim`

Derived:

- CategoryName
- SubcategoryName
- ProductFullHierarchy

---

### 🔹 Customer Dimension Preparation

Joined:

- Customer
- Person
- Territory

Created table:

`customer_dim`

Derived:

- CustomerType
- TerritoryName
- Region
- Country

---

### 🔹 Date Dimension Preparation

Created manually using SQL:

`date_dim`

Columns:

- DateKey (YYYYMMDD)
- FullDate
- Day
- Month
- MonthName
- Quarter
- Year
- WeekNumber
- IsWeekend

---

# 🏛 3️⃣ DWH Layer (Data Warehouse)

## 📌 Purpose

Final analytical layer optimized for reporting and BI tools.

Uses:

- Star Schema
- Surrogate Keys
- Fact & Dimension separation

## 🗄 Database Name

`AdventureWorks_DWH`

---

# ⭐ Star Schema Design

## 🧩 Dimensions

1. DimCustomer  
2. DimProduct  
3. DimDate  
4. DimEmployee  
5. DimTerritory  

Each dimension:

- Uses surrogate primary key (IDENTITY)
- Preserves Business Key
- Contains descriptive attributes
- Applies SCD Type 1 (overwrite strategy)

---

## 📦 Fact Table

### FactSales

Grain:

> One record per SalesOrderDetail line

### Measures:

- OrderQty
- UnitPrice
- LineTotal
- DiscountAmount
- TaxAmount
- TotalDue

### Foreign Keys:

- CustomerKey
- ProductKey
- DateKey
- EmployeeKey
- TerritoryKey

---

# 🔁 ETL Flow in SSIS

## 1️⃣ ODS Load Package

- Extract from AdventureWorks2012
- Load into ODS
- Truncate tables before full load

---

## 2️⃣ STG Load Package

- Extract from ODS
- Apply Derived Column Transformations
- Apply Lookup Transformations
- Load STG dimension preparation tables

---

## 3️⃣ DWH Dimension Load Package

### Slowly Changing Dimension Strategy

- Type 1 for most dimensions (overwrite)
- Surrogate Key generation using IDENTITY
- Lookup to detect existing records
- Conditional Split for insert/update logic

---

## 4️⃣ Fact Load Package

Steps:

1. Extract SalesOrderHeader + SalesOrderDetail from STG
2. Lookup surrogate keys from dimensions
3. Handle missing keys
4. Insert into FactSales

---

# 🛠 Technologies Used

- SQL Server 2022+
- SQL Server Integration Services (SSIS)
- SQL Server Management Studio (SSMS)
- AdventureWorks2012
- T-SQL
- Star Schema Modeling

---

# ▶️ How to Run the Project

## 🔹 Step 1: Install Requirements

- Install SQL Server
- Install SSMS
- Install SSDT (for SSIS development)
- Restore AdventureWorks2012 database

---

## 🔹 Step 2: Create Databases

```sql
CREATE DATABASE AdventureWorks_ODS;
CREATE DATABASE AdventureWorks_STG;
CREATE DATABASE AdventureWorks_DWH;
```
## 🔹 Step 3: Execute Scripts

Run ODS table creation scripts

Run STG table creation scripts

Run DWH dimension & fact creation scripts

## 🔹 Step 4: Open SSIS Solution

Open .sln file in Visual Studio

Update connection managers

Execute packages in order:

ODS_Load.dtsx

STG_Load.dtsx

DWH_Dim_Load.dtsx

DWH_Fact_Load.dtsx

# 🧪 Testing the Warehouse

Example Query:
```
SELECT 
    d.Year,
    p.CategoryName,
    SUM(f.LineTotal) AS TotalSales
FROM FactSales f
JOIN DimDate d ON f.DateKey = d.DateKey
JOIN DimProduct p ON f.ProductKey = p.ProductKey
GROUP BY d.Year, p.CategoryName
ORDER BY d.Year;
```

# 📊 BI Integration

The Data Warehouse can be directly connected to:

Power BI

Tableau

SSRS

Use AdventureWorks_DWH as the data source.

# 🎯 Project Achievements

✔ Full ETL pipeline
✔ Layered architecture (ODS → STG → DWH)
✔ Star Schema modeling
✔ Surrogate key implementation
✔ Fact-Dimension relationships
✔ Production-style SSIS packages

# 🚀 Author

**Abram Ashraf**

Data Engineer | Data Warehouse Developer
Specialized in ETL, SSIS, Data Modeling, and Analytics
