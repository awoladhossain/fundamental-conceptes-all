# Complete SQL Query Guide: From Basics to Advanced

## Table of Contents
1. [Database Setup for Practice](#database-setup-for-practice)
2. [Basic SELECT Statement](#basic-select-statement)
3. [WHERE Clause - Filtering Data](#where-clause---filtering-data)
4. [Operators in SQL](#operators-in-sql)
5. [Sorting with ORDER BY](#sorting-with-order-by)
6. [LIMIT and OFFSET](#limit-and-offset)
7. [DISTINCT - Removing Duplicates](#distinct---removing-duplicates)
8. [Aggregate Functions](#aggregate-functions)
9. [GROUP BY - Grouping Data](#group-by---grouping-data)
10. [HAVING - Filtering Groups](#having---filtering-groups)
11. [WHERE vs HAVING](#where-vs-having)
12. [Subqueries](#subqueries)
13. [CASE Statements](#case-statements)
14. [String Functions](#string-functions)
15. [Date and Time Functions](#date-and-time-functions)
16. [NULL Handling](#null-handling)
17. [UNION, INTERSECT, EXCEPT](#union-intersect-except)
18. [Advanced Query Patterns](#advanced-query-patterns)
19. [Query Execution Order](#query-execution-order)
20. [Complete Practice Examples](#complete-practice-examples)

---

# Database Setup for Practice

Let's create sample tables that we'll use throughout this guide.

```sql
-- Create Employees Table
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY AUTO_INCREMENT,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100),
    Department VARCHAR(50),
    Salary DECIMAL(10, 2),
    HireDate DATE,
    ManagerID INT,
    City VARCHAR(50),
    Country VARCHAR(50)
);

-- Insert Sample Data
INSERT INTO Employees (FirstName, LastName, Email, Department, Salary, HireDate, ManagerID, City, Country) VALUES
('John', 'Smith', 'john.smith@company.com', 'IT', 75000, '2020-01-15', NULL, 'New York', 'USA'),
('Sarah', 'Johnson', 'sarah.j@company.com', 'HR', 65000, '2019-03-20', 1, 'Los Angeles', 'USA'),
('Mike', 'Davis', 'mike.d@company.com', 'IT', 80000, '2021-06-10', 1, 'New York', 'USA'),
('Emily', 'Wilson', 'emily.w@company.com', 'Sales', 70000, '2020-09-05', NULL, 'Chicago', 'USA'),
('David', 'Brown', 'david.b@company.com', 'Sales', 72000, '2018-11-12', 4, 'Chicago', 'USA'),
('Lisa', 'Anderson', 'lisa.a@company.com', 'IT', 78000, '2020-02-28', 1, 'San Francisco', 'USA'),
('James', 'Taylor', 'james.t@company.com', 'HR', 62000, '2021-08-15', 2, 'Boston', 'USA'),
('Anna', 'Martinez', 'anna.m@company.com', 'Marketing', 68000, '2019-05-22', NULL, 'Miami', 'USA'),
('Robert', 'Garcia', 'robert.g@company.com', 'IT', 85000, '2017-07-30', 1, 'Seattle', 'USA'),
('Jennifer', 'Lee', 'jennifer.l@company.com', 'Sales', 71000, '2020-10-18', 4, 'Denver', 'USA');

-- Create Products Table
CREATE TABLE Products (
    ProductID INT PRIMARY KEY AUTO_INCREMENT,
    ProductName VARCHAR(100),
    Category VARCHAR(50),
    Price DECIMAL(10, 2),
    Stock INT,
    SupplierID INT
);

INSERT INTO Products (ProductName, Category, Price, Stock, SupplierID) VALUES
('Laptop Pro', 'Electronics', 1200.00, 50, 1),
('Wireless Mouse', 'Electronics', 25.00, 200, 1),
('Office Desk', 'Furniture', 350.00, 30, 2),
('Ergonomic Chair', 'Furniture', 280.00, 45, 2),
('Monitor 24"', 'Electronics', 180.00, 75, 1),
('Keyboard Mechanical', 'Electronics', 120.00, 100, 1),
('Filing Cabinet', 'Furniture', 150.00, 25, 2),
('Desk Lamp', 'Furniture', 45.00, 150, 3),
('Webcam HD', 'Electronics', 80.00, 60, 1),
('Notebook Set', 'Stationery', 15.00, 300, 3);

-- Create Orders Table
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY AUTO_INCREMENT,
    CustomerName VARCHAR(100),
    ProductID INT,
    Quantity INT,
    OrderDate DATE,
    ShippingCity VARCHAR(50),
    TotalAmount DECIMAL(10, 2),
    Status VARCHAR(20),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);

INSERT INTO Orders (CustomerName, ProductID, Quantity, OrderDate, ShippingCity, TotalAmount, Status) VALUES
('Alice Cooper', 1, 2, '2024-01-15', 'New York', 2400.00, 'Delivered'),
('Bob Martin', 2, 5, '2024-01-16', 'Los Angeles', 125.00, 'Delivered'),
('Charlie Brown', 3, 1, '2024-01-17', 'Chicago', 350.00, 'Shipped'),
('Diana Prince', 1, 1, '2024-01-18', 'Houston', 1200.00, 'Delivered'),
('Eve Adams', 5, 2, '2024-01-19', 'Phoenix', 360.00, 'Processing'),
('Frank Castle', 4, 3, '2024-01-20', 'Philadelphia', 840.00, 'Delivered'),
('Grace Lee', 2, 10, '2024-01-21', 'San Antonio', 250.00, 'Shipped'),
('Henry Ford', 6, 2, '2024-01-22', 'San Diego', 240.00, 'Delivered'),
('Iris West', 7, 1, '2024-01-23', 'Dallas', 150.00, 'Processing'),
('Jack Ryan', 8, 5, '2024-01-24', 'Austin', 225.00, 'Delivered'),
('Alice Cooper', 5, 3, '2024-01-25', 'New York', 540.00, 'Shipped'),
('Bob Martin', 9, 1, '2024-01-26', 'Los Angeles', 80.00, 'Processing');
```

---

# Basic SELECT Statement

## **What is SELECT?**
SELECT is the **most fundamental SQL command**. It retrieves data from a database table.

### **Basic Syntax:**
```sql
SELECT column1, column2, ...
FROM table_name;
```

### **Think of it as:**
"Show me specific information from this table"

---

## **1. Select All Columns**

```sql
SELECT * FROM Employees;
```

**What does `*` mean?**
- `*` = "ALL columns"
- Shows every column in the table

**Result:** (All 10 rows with all columns)
| EmployeeID | FirstName | LastName | Email | Department | Salary | HireDate | ManagerID | City | Country |
|------------|-----------|----------|-------|------------|--------|----------|-----------|------|---------|
| 1 | John | Smith | john.smith@company.com | IT | 75000 | 2020-01-15 | NULL | New York | USA |
| 2 | Sarah | Johnson | sarah.j@company.com | HR | 65000 | 2019-03-20 | 1 | Los Angeles | USA |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

**When to use:** 
- ✅ Quick data exploration
- ✅ Small tables
- ❌ Production queries (slow and wastes resources)

---

## **2. Select Specific Columns**

```sql
SELECT FirstName, LastName, Department 
FROM Employees;
```

**Result:**
| FirstName | LastName | Department |
|-----------|----------|------------|
| John | Smith | IT |
| Sarah | Johnson | HR |
| Mike | Davis | IT |
| Emily | Wilson | Sales |
| ... | ... | ... |

**Why select specific columns?**
- ✅ Faster (less data to transfer)
- ✅ Clearer (shows only what you need)
- ✅ Better performance

---

## **3. Column Aliases (Renaming Columns)**

```sql
SELECT 
    FirstName AS 'First Name',
    LastName AS 'Last Name',
    Salary AS 'Annual Salary',
    Department AS 'Dept'
FROM Employees;
```

**Result:**
| First Name | Last Name | Annual Salary | Dept |
|------------|-----------|---------------|------|
| John | Smith | 75000 | IT |
| Sarah | Johnson | 65000 | HR |
| ... | ... | ... | ... |

**When to use:**
- Make column names more readable
- Rename calculated columns
- Match specific output format requirements

---

## **4. Calculated Columns**

```sql
SELECT 
    FirstName,
    LastName,
    Salary,
    Salary * 12 AS AnnualSalary,
    Salary * 0.10 AS MonthlyBonus
FROM Employees;
```

**Result:**
| FirstName | LastName | Salary | AnnualSalary | MonthlyBonus |
|-----------|----------|--------|--------------|--------------|
| John | Smith | 75000 | 900000 | 7500 |
| Sarah | Johnson | 65000 | 780000 | 6500 |
| ... | ... | ... | ... | ... |

**You can use:**
- Math operators: `+`, `-`, `*`, `/`, `%`
- Functions: `CONCAT()`, `UPPER()`, `ROUND()`, etc.

---

## **5. Combining Text (String Concatenation)**

```sql
SELECT 
    CONCAT(FirstName, ' ', LastName) AS FullName,
    CONCAT(City, ', ', Country) AS Location,
    CONCAT('$', Salary) AS FormattedSalary
FROM Employees;
```

**Result:**
| FullName | Location | FormattedSalary |
|----------|----------|-----------------|
| John Smith | New York, USA | $75000 |
| Sarah Johnson | Los Angeles, USA | $65000 |
| ... | ... | ... |

---

# WHERE Clause - Filtering Data

## **What is WHERE?**
WHERE filters rows based on conditions. Only rows that meet the condition are returned.

### **Syntax:**
```sql
SELECT columns
FROM table
WHERE condition;
```

### **Think of it as:**
"Show me data WHERE some condition is TRUE"

---

## **1. Simple Equality Condition**

```sql
SELECT * 
FROM Employees 
WHERE Department = 'IT';
```

**Result:** Only IT department employees
| EmployeeID | FirstName | LastName | Department | Salary |
|------------|-----------|----------|------------|--------|
| 1 | John | Smith | IT | 75000 |
| 3 | Mike | Davis | IT | 80000 |
| 6 | Lisa | Anderson | IT | 78000 |
| 9 | Robert | Garcia | IT | 85000 |

**Important:** 
- Text values use **single quotes**: `'IT'`
- Numbers **no quotes**: `Salary = 75000`
- SQL is **case-insensitive** for keywords, but **case-sensitive** for data values (depends on database)

---

## **2. Numeric Comparisons**

```sql
-- Salary greater than 70000
SELECT FirstName, LastName, Salary 
FROM Employees 
WHERE Salary > 70000;
```

**Result:**
| FirstName | LastName | Salary |
|-----------|----------|--------|
| John | Smith | 75000 |
| Mike | Davis | 80000 |
| David | Brown | 72000 |
| Lisa | Anderson | 78000 |
| Robert | Garcia | 85000 |
| Jennifer | Lee | 71000 |

---

```sql
-- Salary less than or equal to 70000
SELECT FirstName, LastName, Salary 
FROM Employees 
WHERE Salary <= 70000;
```

**Result:**
| FirstName | LastName | Salary |
|-----------|----------|--------|
| Sarah | Johnson | 65000 |
| Emily | Wilson | 70000 |
| James | Taylor | 62000 |
| Anna | Martinez | 68000 |

---

## **3. Date Comparisons**

```sql
-- Employees hired after 2020
SELECT FirstName, LastName, HireDate 
FROM Employees 
WHERE HireDate > '2020-01-01';
```

**Result:**
| FirstName | LastName | HireDate |
|-----------|----------|----------|
| Mike | Davis | 2021-06-10 |
| Lisa | Anderson | 2020-02-28 |
| James | Taylor | 2021-08-15 |
| Jennifer | Lee | 2020-10-18 |

**Date Format:** `'YYYY-MM-DD'` (e.g., '2020-01-01')

---

## **4. NOT EQUAL**

```sql
-- All employees NOT in IT department
SELECT FirstName, LastName, Department 
FROM Employees 
WHERE Department != 'IT';
-- OR: WHERE Department <> 'IT';  (alternative syntax)
```

**Result:**
| FirstName | LastName | Department |
|-----------|----------|------------|
| Sarah | Johnson | HR |
| Emily | Wilson | Sales |
| David | Brown | Sales |
| James | Taylor | HR |
| Anna | Martinez | Marketing |
| Jennifer | Lee | Sales |

---

# Operators in SQL

## **Comparison Operators**

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal to | `WHERE Salary = 75000` |
| `!=` or `<>` | Not equal to | `WHERE Department != 'IT'` |
| `>` | Greater than | `WHERE Salary > 70000` |
| `<` | Less than | `WHERE Salary < 70000` |
| `>=` | Greater than or equal | `WHERE Salary >= 70000` |
| `<=` | Less than or equal | `WHERE Salary <= 70000` |

---

## **Logical Operators**

### **1. AND - All conditions must be TRUE**

```sql
SELECT FirstName, LastName, Department, Salary 
FROM Employees 
WHERE Department = 'IT' 
  AND Salary > 75000;
```

**Logic:**
- Department MUST be 'IT' **AND**
- Salary MUST be greater than 75000
- **Both conditions must be true**

**Result:**
| FirstName | LastName | Department | Salary |
|-----------|----------|------------|--------|
| Mike | Davis | IT | 80000 |
| Lisa | Anderson | IT | 78000 |
| Robert | Garcia | IT | 85000 |

---

### **2. OR - At least one condition must be TRUE**

```sql
SELECT FirstName, LastName, Department 
FROM Employees 
WHERE Department = 'IT' 
   OR Department = 'Sales';
```

**Logic:**
- Department can be 'IT' **OR**
- Department can be 'Sales'
- **At least one condition must be true**

**Result:**
| FirstName | LastName | Department |
|-----------|----------|------------|
| John | Smith | IT |
| Mike | Davis | IT |
| Emily | Wilson | Sales |
| David | Brown | Sales |
| Lisa | Anderson | IT |
| Robert | Garcia | IT |
| Jennifer | Lee | Sales |

---

### **3. NOT - Negates a condition**

```sql
SELECT FirstName, LastName, Department 
FROM Employees 
WHERE NOT Department = 'IT';
-- Same as: WHERE Department != 'IT'
```

**Result:** All employees NOT in IT

---

### **4. Combining AND, OR, NOT**

```sql
SELECT FirstName, LastName, Department, Salary 
FROM Employees 
WHERE (Department = 'IT' OR Department = 'Sales') 
  AND Salary > 70000;
```

**Logic:**
1. First evaluate `(Department = 'IT' OR Department = 'Sales')`
2. Then check if `Salary > 70000`
3. **Use parentheses to control order!**

**Result:**
| FirstName | LastName | Department | Salary |
|-----------|----------|------------|--------|
| John | Smith | IT | 75000 |
| Mike | Davis | IT | 80000 |
| David | Brown | Sales | 72000 |
| Lisa | Anderson | IT | 78000 |
| Robert | Garcia | IT | 85000 |
| Jennifer | Lee | Sales | 71000 |

---

## **BETWEEN Operator**

### **Syntax:**
```sql
WHERE column BETWEEN value1 AND value2
```

**Inclusive:** Includes both value1 and value2

```sql
-- Salaries between 65000 and 75000 (inclusive)
SELECT FirstName, LastName, Salary 
FROM Employees 
WHERE Salary BETWEEN 65000 AND 75000;
```

**Result:**
| FirstName | LastName | Salary |
|-----------|----------|--------|
| John | Smith | 75000 |
| Sarah | Johnson | 65000 |
| Emily | Wilson | 70000 |
| David | Brown | 72000 |
| Anna | Martinez | 68000 |
| Jennifer | Lee | 71000 |

**Same as:**
```sql
WHERE Salary >= 65000 AND Salary <= 75000
```

**Date Range:**
```sql
SELECT FirstName, LastName, HireDate 
FROM Employees 
WHERE HireDate BETWEEN '2019-01-01' AND '2020-12-31';
```

---

## **IN Operator**

### **What is IN?**
Checks if a value matches ANY value in a list. Shortcut for multiple OR conditions.

```sql
SELECT FirstName, LastName, Department 
FROM Employees 
WHERE Department IN ('IT', 'Sales', 'HR');
```

**Same as:**
```sql
WHERE Department = 'IT' 
   OR Department = 'Sales' 
   OR Department = 'HR'
```

**Result:**
| FirstName | LastName | Department |
|-----------|----------|------------|
| John | Smith | IT |
| Sarah | Johnson | HR |
| Mike | Davis | IT |
| Emily | Wilson | Sales |
| David | Brown | Sales |
| Lisa | Anderson | IT |
| James | Taylor | HR |
| Robert | Garcia | IT |
| Jennifer | Lee | Sales |

**NOT IN:**
```sql
SELECT FirstName, LastName, Department 
FROM Employees 
WHERE Department NOT IN ('IT', 'Sales');
```

**Result:** Only HR and Marketing employees

---

## **LIKE Operator - Pattern Matching**

### **What is LIKE?**
Used for pattern matching with wildcards.

### **Wildcards:**
- `%` = Zero or more characters
- `_` = Exactly one character

---

### **1. Starts With**

```sql
-- Find names starting with 'J'
SELECT FirstName, LastName 
FROM Employees 
WHERE FirstName LIKE 'J%';
```

**Pattern:** `J%` = J followed by anything
**Result:**
| FirstName | LastName |
|-----------|----------|
| John | Smith |
| James | Taylor |
| Jennifer | Lee |

---

### **2. Ends With**

```sql
-- Find emails ending with 'company.com'
SELECT FirstName, Email 
FROM Employees 
WHERE Email LIKE '%@company.com';
```

**Pattern:** `%@company.com` = anything followed by @company.com
**Result:** All employees (all emails end with @company.com)

---

### **3. Contains**

```sql
-- Find names containing 'son'
SELECT FirstName, LastName 
FROM Employees 
WHERE LastName LIKE '%son%';
```

**Pattern:** `%son%` = anything + "son" + anything
**Result:**
| FirstName | LastName |
|-----------|----------|
| Sarah | Johnson |
| Lisa | Anderson |

---

### **4. Specific Position**

```sql
-- Find names with 'a' as second letter
SELECT FirstName 
FROM Employees 
WHERE FirstName LIKE '_a%';
```

**Pattern:** 
- `_` = any one character
- `a` = the letter 'a'
- `%` = anything after

**Result:**
| FirstName |
|-----------|
| Sarah |
| James |
| David |

---

### **5. Exact Length**

```sql
-- Find first names exactly 4 characters long
SELECT FirstName 
FROM Employees 
WHERE FirstName LIKE '____';  -- Four underscores
```

**Pattern:** `____` = exactly 4 characters

**Result:**
| FirstName |
|-----------|
| John |
| Mike |
| Lisa |
| Anna |

---

### **6. Case Sensitivity**

```sql
-- Case-insensitive (most databases)
WHERE FirstName LIKE 'john%';  -- Matches 'John', 'JOHN', 'john'

-- Case-sensitive (if needed - MySQL)
WHERE FirstName LIKE BINARY 'john%';  -- Matches only 'john'
```

---

## **IS NULL and IS NOT NULL**

### **What is NULL?**
NULL means "no value" or "unknown" - it's NOT the same as empty string '' or 0.

```sql
-- Find employees without a manager (top-level employees)
SELECT FirstName, LastName, ManagerID 
FROM Employees 
WHERE ManagerID IS NULL;
```

**Result:**
| FirstName | LastName | ManagerID |
|-----------|----------|-----------|
| John | Smith | NULL |
| Emily | Wilson | NULL |
| Anna | Martinez | NULL |

**Important:** 
- ❌ `WHERE ManagerID = NULL` (WRONG - doesn't work!)
- ✅ `WHERE ManagerID IS NULL` (CORRECT)

---

```sql
-- Find employees WITH a manager
SELECT FirstName, LastName, ManagerID 
FROM Employees 
WHERE ManagerID IS NOT NULL;
```

**Result:** All employees who have a manager (ManagerID is not NULL)

---

# Sorting with ORDER BY

## **What is ORDER BY?**
Sorts the result set by one or more columns.

### **Syntax:**
```sql
SELECT columns
FROM table
ORDER BY column1 [ASC|DESC], column2 [ASC|DESC];
```

- **ASC** = Ascending (smallest to largest) - DEFAULT
- **DESC** = Descending (largest to smallest)

---

## **1. Simple Sort - Ascending**

```sql
-- Sort by salary (lowest to highest)
SELECT FirstName, LastName, Salary 
FROM Employees 
ORDER BY Salary;
-- OR: ORDER BY Salary ASC;
```

**Result:**
| FirstName | LastName | Salary |
|-----------|----------|--------|
| James | Taylor | 62000 |
| Sarah | Johnson | 65000 |
| Anna | Martinez | 68000 |
| Emily | Wilson | 70000 |
| Jennifer | Lee | 71000 |
| David | Brown | 72000 |
| John | Smith | 75000 |
| Lisa | Anderson | 78000 |
| Mike | Davis | 80000 |
| Robert | Garcia | 85000 |

---

## **2. Sort Descending**

```sql
-- Sort by salary (highest to lowest)
SELECT FirstName, LastName, Salary 
FROM Employees 
ORDER BY Salary DESC;
```

**Result:**
| FirstName | LastName | Salary |
|-----------|----------|--------|
| Robert | Garcia | 85000 |
| Mike | Davis | 80000 |
| Lisa | Anderson | 78000 |
| John | Smith | 75000 |
| ... | ... | ... |

---

## **3. Sort by Multiple Columns**

```sql
-- Sort by Department, then by Salary within each department
SELECT FirstName, LastName, Department, Salary 
FROM Employees 
ORDER BY Department ASC, Salary DESC;
```

**Logic:**
1. First, group by Department (alphabetically)
2. Within each department, sort by Salary (highest first)

**Result:**
| FirstName | LastName | Department | Salary |
|-----------|----------|------------|--------|
| Sarah | Johnson | HR | 65000 |
| James | Taylor | HR | 62000 |
| Robert | Garcia | IT | 85000 |
| Mike | Davis | IT | 80000 |
| Lisa | Anderson | IT | 78000 |
| John | Smith | IT | 75000 |
| Anna | Martinez | Marketing | 68000 |
| David | Brown | Sales | 72000 |
| Jennifer | Lee | Sales | 71000 |
| Emily | Wilson | Sales | 70000 |

---

## **4. Sort by Column Position**

```sql
-- Sort by the 3rd column in SELECT list (Department)
SELECT FirstName, LastName, Department, Salary 
FROM Employees 
ORDER BY 3, 4 DESC;
-- Same as: ORDER BY Department, Salary DESC;
```

**Note:** While this works, it's **not recommended** (hard to read and maintain).

---

## **5. Sort with Calculated Columns**

```sql
SELECT 
    FirstName, 
    LastName, 
    Salary,
    Salary * 12 AS AnnualSalary 
FROM Employees 
ORDER BY AnnualSalary DESC;
```

**Result:** Sorted by annual salary (highest to lowest)

---

## **6. Sort by Text Columns**

```sql
-- Alphabetical by last name
SELECT FirstName, LastName 
FROM Employees 
ORDER BY LastName;
```

**Result:**
| FirstName | LastName |
|-----------|----------|
| Lisa | Anderson |
| David | Brown |
| Mike | Davis |
| Robert | Garcia |
| Sarah | Johnson |
| Jennifer | Lee |
| Anna | Martinez |
| John | Smith |
| James | Taylor |
| Emily | Wilson |

---

# LIMIT and OFFSET

## **LIMIT - Restrict Number of Rows**

### **Syntax:**
```sql
SELECT columns
FROM table
ORDER BY column
LIMIT number;
```

---

## **1. Get Top N Records**

```sql
-- Get top 5 highest paid employees
SELECT FirstName, LastName, Salary 
FROM Employees 
ORDER BY Salary DESC 
LIMIT 5;
```

**Result:**
| FirstName | LastName | Salary |
|-----------|----------|--------|
| Robert | Garcia | 85000 |
| Mike | Davis | 80000 |
| Lisa | Anderson | 78000 |
| John | Smith | 75000 |
| David | Brown | 72000 |

---

## **2. Get Bottom N Records**

```sql
-- Get 3 lowest paid employees
SELECT FirstName, LastName, Salary 
FROM Employees 
ORDER BY Salary ASC 
LIMIT 3;
```

**Result:**
| FirstName | LastName | Salary |
|-----------|----------|--------|
| James | Taylor | 62000 |
| Sarah | Johnson | 65000 |
| Anna | Martinez | 68000 |

---

## **OFFSET - Skip Rows**

### **Syntax:**
```sql
SELECT columns
FROM table
ORDER BY column
LIMIT number OFFSET skip_count;
```

---

## **Pagination Example**

```sql
-- Page 1: First 3 employees
SELECT FirstName, LastName, Salary 
FROM Employees 
ORDER BY Salary DESC 
LIMIT 3 OFFSET 0;
```

**Result (Page 1):**
| FirstName | LastName | Salary |
|-----------|----------|--------|
| Robert | Garcia | 85000 |
| Mike | Davis | 80000 |
| Lisa | Anderson | 78000 |

---

```sql
-- Page 2: Next 3 employees (skip first 3)
SELECT FirstName, LastName, Salary 
FROM Employees 
ORDER BY Salary DESC 
LIMIT 3 OFFSET 3;
```

**Result (Page 2):**
| FirstName | LastName | Salary |
|-----------|----------|--------|
| John | Smith | 75000 |
| David | Brown | 72000 |
| Jennifer | Lee | 71000 |

---

```sql
-- Page 3: Next 3 employees (skip first 6)
SELECT FirstName, LastName, Salary 
FROM Employees 
ORDER BY Salary DESC 
LIMIT 3 OFFSET 6;
```

**Pagination Formula:**
```
Page 1: LIMIT 3 OFFSET 0   (skip 0)
Page 2: LIMIT 3 OFFSET 3   (skip 3)
Page 3: LIMIT 3 OFFSET 6   (skip 6)
Page N: LIMIT 3 OFFSET (N-1)*3
```

---

## **Alternative Syntax (MySQL)**

```sql
-- LIMIT offset, count
SELECT FirstName, LastName, Salary 
FROM Employees 
ORDER BY Salary DESC 
LIMIT 3, 3;  -- Skip 3, show 3
```

---

# DISTINCT - Removing Duplicates

## **What is DISTINCT?**
Returns only unique (different) values, removing duplicates.

### **Syntax:**
```sql
SELECT DISTINCT column1, column2
FROM table;
```

---

## **1. Single Column DISTINCT**

```sql
-- Get all unique departments
SELECT DISTINCT Department 
FROM Employees;
```

**Without DISTINCT:**
```
IT, HR, IT, Sales, Sales, IT, HR, Marketing, IT, Sales
```

**With DISTINCT:**
| Department |
|------------|
| IT |
| HR |
| Sales |
| Marketing |

---

## **2. Multiple Columns DISTINCT**

```sql
-- Get unique combinations of Department and City
SELECT DISTINCT Department, City 
FROM Employees;
```

**Result:** Unique pairs of (Department, City)
| Department | City |
|------------|------|
| IT | New York |
| HR | Los Angeles |
| IT | San Francisco |
| Sales | Chicago |
| ... | ... |

**Important:** DISTINCT applies to the **entire row**, not individual columns!

---

## **3. Count Distinct Values**

```sql
-- How many different departments exist?
SELECT COUNT(DISTINCT Department) AS UniqueDepartments 
FROM Employees;
```

**Result:**
| UniqueDepartments |
|-------------------|
| 4 |

---

## **4. DISTINCT with ORDER BY**

```sql
SELECT DISTINCT Department 
FROM Employees 
ORDER BY Department;
```

**Result:** Unique departments sorted alphabetically
| Department |
|------------|
| HR |
| IT |
| Marketing |
| Sales |

---

# Aggregate Functions

## **What are Aggregate Functions?**
Functions that perform calculations on a set of values and return a **single value**.

### **Common Aggregate Functions:**
- `COUNT()` - Count rows
- `SUM()` - Add up values
- `AVG()` - Calculate average
- `MIN()` - Find minimum value
- `MAX()` - Find maximum value

---

## **1. COUNT - Count Rows**

### **Count all rows:**
```sql
-- How many employees total?
SELECT COUNT(*) AS TotalEmployees 
FROM Employees;
```

**Result:**
| TotalEmployees |
|----------------|
| 10 |

---

### **Count non-NULL values:**
```sql
-- How many employees have a manager?
SELECT COUNT(ManagerID) AS EmployeesWithManager 
FROM Employees;
```

**Result:**
| EmployeesWithManager |
|----------------------|
| 7 |

**Difference:**
- `COUNT(*)` counts ALL rows (including NULLs)
- `COUNT(column)` counts only non-NULL values in that column

---

### **Count distinct:**
```sql
-- How many different departments?
SELECT COUNT(DISTINCT Department) AS UniqueDepartments 
FROM Employees;
```

**Result:**
| UniqueDepartments |
|-------------------|
| 4 |

---

## **2. SUM - Add Up Values**

```sql
-- Total salary expenditure
SELECT SUM(Salary) AS TotalSalaries 
FROM Employees;
```

**Result:**
| TotalSalaries |
|---------------|
| 726000 |

**Calculation:** 75000 + 65000 + 80000 + 70000 + 72000 + 78000 + 62000 + 68000 + 85000 + 71000 = 726000

---

### **SUM with WHERE:**
```sql
-- Total IT department salaries
SELECT SUM(Salary) AS ITSalaries 
FROM Employees 
WHERE Department = 'IT';
```

**Result:**
| ITSalaries |
|------------|
| 318000 |

---

## **3. AVG - Calculate Average**

```sql
-- Average salary across all employees
SELECT AVG(Salary) AS AverageSalary 
FROM Employees;
```

**Result:**
| AverageSalary |
|---------------|
| 72600 |

**Calculation:** 726000 / 10 = 72600

---

### **AVG with ROUND:**
```sql
SELECT ROUND(AVG(Salary), 2) AS AverageSalary 
FROM Employees;
```

**ROUND(value, decimal_places)** - Rounds to specified decimal places

---

### **AVG by Department:**
```sql
-- Average salary in IT department
SELECT AVG(Salary) AS ITAverageSalary 
FROM Employees 
WHERE Department = 'IT';
```

**Result:**
| ITAverageSalary |
|-----------------|
| 79500 |

---

## **4. MIN - Find Minimum Value**

```sql
-- Lowest salary
SELECT MIN(Salary) AS LowestSalary 
FROM Employees;
```

**Result:**
| LowestSalary |
|--------------|
| 62000 |

---

### **MIN with other columns:**
```sql
-- Who has the lowest salary?
SELECT FirstName, LastName, Salary 
FROM Employees 
WHERE Salary = (SELECT MIN(Salary) FROM Employees);
```

**Result:**
| FirstName | LastName | Salary |
|-----------|----------|--------|
| James | Taylor | 62000 |

---

## **5. MAX - Find Maximum Value**

```sql
-- Highest salary
SELECT MAX(Salary) AS HighestSalary 
FROM Employees;
```

**Result:**
| HighestSalary |
|---------------|
| 85000 |

---

### **Find earliest and latest hire dates:**
```sql
SELECT 
    MIN(HireDate) AS FirstHireDate,
    MAX(HireDate) AS LatestHireDate 
FROM Employees;
```

**Result:**
| FirstHireDate | LatestHireDate |
|---------------|----------------|
| 2017-07-30 | 2021-08-15 |

---

## **6. Multiple Aggregates Together**

```sql
SELECT 
    COUNT(*) AS TotalEmployees,
    AVG(Salary) AS AverageSalary,
    MIN(Salary) AS LowestSalary,
    MAX(Salary) AS HighestSalary,
    SUM(Salary) AS TotalPayroll
FROM Employees;
```

**Result:**
| TotalEmployees | AverageSalary | LowestSalary | HighestSalary | TotalPayroll |
|----------------|---------------|--------------|---------------|--------------|
| 10 | 72600 | 62000 | 85000 | 726000 |

---

# GROUP BY - Grouping Data

## **What is GROUP BY?**
Groups rows that have the same values in specified columns, allowing you to perform aggregate functions on each group.

### **Syntax:**
```sql
SELECT column1, AGGREGATE_FUNCTION(column2)
FROM table
GROUP BY column1;
```

### **Think of it as:**
"Split data into groups and calculate statistics for each group"

---

## **1. Simple GROUP BY**

```sql
-- Count employees in each department
SELECT 
    Department,
    COUNT(*) AS EmployeeCount
FROM Employees 
GROUP BY Department;
```

**What happens:**
1. Split employees by Department
2. Count rows in each group

**Result:**
| Department | EmployeeCount |
|------------|---------------|
| HR | 2 |
| IT | 4 |
| Marketing | 1 |
| Sales | 3 |

---

## **2. GROUP BY with Multiple Aggregates**

```sql
-- Department statistics
SELECT 
    Department,
    COUNT(*) AS EmployeeCount,
    AVG(Salary) AS AverageSalary,
    MIN(Salary) AS MinSalary,
    MAX(Salary) AS MaxSalary,
    SUM(Salary) AS TotalSalary
FROM Employees 
GROUP BY Department;
```

**Result:**
| Department | EmployeeCount | AverageSalary | MinSalary | MaxSalary | TotalSalary |
|------------|---------------|---------------|-----------|-----------|-------------|
| HR | 2 | 63500 | 62000 | 65000 | 127000 |
| IT | 4 | 79500 | 75000 | 85000 | 318000 |
| Marketing | 1 | 68000 | 68000 | 68000 | 68000 |
| Sales | 3 | 71000 | 70000 | 72000 | 213000 |

---

## **3. GROUP BY Multiple Columns**

```sql
-- Employee count by Department and City
SELECT 
    Department,
    City,
    COUNT(*) AS EmployeeCount
FROM Employees 
GROUP BY Department, City
ORDER BY Department, City;
```

**Result:**
| Department | City | EmployeeCount |
|------------|------|---------------|
| HR | Boston | 1 |
| HR | Los Angeles | 1 |
| IT | New York | 2 |
| IT | San Francisco | 1 |
| IT | Seattle | 1 |
| Marketing | Miami | 1 |
| Sales | Chicago | 2 |
| Sales | Denver | 1 |

**What happens:**
- Groups by unique combinations of (Department, City)
- Counts employees in each combination

---

## **4. GROUP BY with WHERE**

```sql
-- Average salary by department for employees hired after 2019
SELECT 
    Department,
    AVG(Salary) AS AverageSalary
FROM Employees 
WHERE HireDate > '2019-01-01'
GROUP BY Department;
```

**Execution Order:**
1. **WHERE** filters rows FIRST (hired after 2019)
2. **GROUP BY** groups the filtered rows
3. **AVG** calculates average for each group

**Result:**
| Department | AverageSalary |
|------------|---------------|
| HR | 63500 |
| IT | 77666.67 |
| Sales | 71000 |

---

## **5. GROUP BY with Calculated Columns**

```sql
-- Salary ranges (group by salary bracket)
SELECT 
    CASE 
        WHEN Salary < 65000 THEN 'Low'
        WHEN Salary BETWEEN 65000 AND 75000 THEN 'Medium'
        ELSE 'High'
    END AS SalaryBracket,
    COUNT(*) AS EmployeeCount,
    AVG(Salary) AS AverageSalary
FROM Employees 
GROUP BY SalaryBracket;
```

**Result:**
| SalaryBracket | EmployeeCount | AverageSalary |
|---------------|---------------|---------------|
| Low | 1 | 62000 |
| Medium | 5 | 69600 |
| High | 4 | 79750 |

---

## **Important GROUP BY Rules:**

### **Rule 1: SELECT columns must be in GROUP BY or be aggregated**

```sql
-- ❌ WRONG - FirstName not in GROUP BY or aggregated
SELECT FirstName, Department, AVG(Salary) 
FROM Employees 
GROUP BY Department;

-- ✅ CORRECT - Only grouped column and aggregate
SELECT Department, AVG(Salary) 
FROM Employees 
GROUP BY Department;

-- ✅ CORRECT - All non-aggregated columns in GROUP BY
SELECT Department, City, AVG(Salary) 
FROM Employees 
GROUP BY Department, City;
```

---

# HAVING - Filtering Groups

## **What is HAVING?**
HAVING filters **groups** (after GROUP BY), while WHERE filters **rows** (before GROUP BY).

### **Syntax:**
```sql
SELECT column1, AGGREGATE_FUNCTION(column2)
FROM table
WHERE condition              -- Filters rows
GROUP BY column1
HAVING condition;            -- Filters groups
```

### **Think of it as:**
- **WHERE** = "Filter individual rows before grouping"
- **HAVING** = "Filter groups after aggregating"

---

## **1. Simple HAVING**

```sql
-- Departments with more than 2 employees
SELECT 
    Department,
    COUNT(*) AS EmployeeCount
FROM Employees 
GROUP BY Department
HAVING COUNT(*) > 2;
```

**What happens:**
1. Group by Department
2. Count employees in each group
3. **Filter out groups** with count ≤ 2

**Result:**
| Department | EmployeeCount |
|------------|---------------|
| IT | 4 |
| Sales | 3 |

**Without HAVING:** Would show all 4 departments (HR: 2, IT: 4, Marketing: 1, Sales: 3)

---

## **2. HAVING with AVG**

```sql
-- Departments with average salary > 70000
SELECT 
    Department,
    AVG(Salary) AS AverageSalary,
    COUNT(*) AS EmployeeCount
FROM Employees 
GROUP BY Department
HAVING AVG(Salary) > 70000;
```

**Result:**
| Department | AverageSalary | EmployeeCount |
|------------|---------------|---------------|
| IT | 79500 | 4 |
| Sales | 71000 | 3 |

---

## **3. HAVING with Multiple Conditions**

```sql
-- Departments with >2 employees AND average salary >70000
SELECT 
    Department,
    COUNT(*) AS EmployeeCount,
    AVG(Salary) AS AverageSalary
FROM Employees 
GROUP BY Department
HAVING COUNT(*) > 2 
   AND AVG(Salary) > 70000;
```

**Result:**
| Department | EmployeeCount | AverageSalary |
|------------|---------------|---------------|
| IT | 4 | 79500 |
| Sales | 3 | 71000 |

---

## **4. WHERE and HAVING Together**

```sql
-- Cities with >1 IT employee hired after 2019
SELECT 
    City,
    COUNT(*) AS ITEmployees
FROM Employees 
WHERE Department = 'IT'           -- Filter rows first
  AND HireDate > '2019-01-01'
GROUP BY City
HAVING COUNT(*) > 1;              -- Filter groups after
```

**Execution Order:**
1. **WHERE**: Filter to IT employees hired after 2019
2. **GROUP BY**: Group by City
3. **HAVING**: Keep only cities with >1 employee

**Result:**
| City | ITEmployees |
|------|-------------|
| New York | 2 |

---

# WHERE vs HAVING

## **Key Differences:**

| Aspect | WHERE | HAVING |
|--------|-------|--------|
| **Filters** | Individual rows | Groups |
| **When** | Before GROUP BY | After GROUP BY |
| **Works with** | Column values | Aggregate functions |
| **Cannot use** | Aggregate functions | - |

---

## **Comparison Examples:**

### **Example 1: Filter rows vs filter groups**

```sql
-- WHERE: Employees with salary > 70000 (individual rows)
SELECT Department, FirstName, Salary
FROM Employees 
WHERE Salary > 70000;
```

**Result:** 6 individual employees

---

```sql
-- HAVING: Departments with average salary > 70000 (groups)
SELECT Department, AVG(Salary) AS AvgSalary
FROM Employees 
GROUP BY Department
HAVING AVG(Salary) > 70000;
```

**Result:** 2 departments (IT and Sales)

---

### **Example 2: Combining WHERE and HAVING**

```sql
SELECT 
    Department,
    COUNT(*) AS EmployeeCount,
    AVG(Salary) AS AvgSalary
FROM Employees 
WHERE HireDate > '2019-01-01'     -- Filter: Only recent hires
GROUP BY Department
HAVING COUNT(*) >= 2;             -- Filter: Only depts with 2+ recent hires
```

**Execution:**
1. WHERE: Keep only employees hired after 2019 (7 rows)
2. GROUP BY: Group these 7 rows by department
3. HAVING: Keep only groups with 2+ employees

---

## **Common Mistake:**

```sql
-- ❌ WRONG - Cannot use aggregate in WHERE
SELECT Department, AVG(Salary)
FROM Employees 
WHERE AVG(Salary) > 70000      -- ERROR!
GROUP BY Department;

-- ✅ CORRECT - Use HAVING for aggregates
SELECT Department, AVG(Salary)
FROM Employees 
GROUP BY Department
HAVING AVG(Salary) > 70000;
```

---

# Subqueries

## **What is a Subquery?**
A query **nested inside another query**. Also called "inner query" or "nested query".

### **Types:**
1. **Scalar Subquery** - Returns single value
2. **Column Subquery** - Returns single column
3. **Row Subquery** - Returns single row
4. **Table Subquery** - Returns multiple rows/columns

---

## **1. Scalar Subquery (Single Value)**

```sql
-- Find employees earning more than average salary
SELECT FirstName, LastName, Salary
FROM Employees 
WHERE Salary > (SELECT AVG(Salary) FROM Employees);
```

**What happens:**
1. Inner query: `SELECT AVG(Salary) FROM Employees` returns 72600
2. Outer query: `WHERE Salary > 72600`

**Result:**
| FirstName | LastName | Salary |
|-----------|----------|--------|
| John | Smith | 75000 |
| Mike | Davis | 80000 |
| Lisa | Anderson | 78000 |
| Robert | Garcia | 85000 |

---

## **2. Subquery with IN**

```sql
-- Find employees in departments that have >2 people
SELECT FirstName, LastName, Department
FROM Employees 
WHERE Department IN (
    SELECT Department 
    FROM Employees 
    GROUP BY Department 
    HAVING COUNT(*) > 2
);
```

**What happens:**
1. Inner query: Returns departments with >2 employees (IT, Sales)
2. Outer query: Get employees in those departments

**Result:**
| FirstName | LastName | Department |
|-----------|----------|------------|
| John | Smith | IT |
| Mike | Davis | IT |
| Emily | Wilson | Sales |
| David | Brown | Sales |
| Lisa | Anderson | IT |
| Robert | Garcia | IT |
| Jennifer | Lee | Sales |

---

## **3. Subquery in FROM (Derived Table)**

```sql
-- Get department statistics and filter
SELECT *
FROM (
    SELECT 
        Department,
        AVG(Salary) AS AvgSalary,
        COUNT(*) AS EmpCount
    FROM Employees 
    GROUP BY Department
) AS DeptStats
WHERE AvgSalary > 70000;
```

**What happens:**
1. Inner query: Creates temporary table with department stats
2. Outer query: Filters this temporary table

**Result:**
| Department | AvgSalary | EmpCount |
|------------|-----------|----------|
| IT | 79500 | 4 |
| Sales | 71000 | 3 |

---

## **4. Correlated Subquery**

A correlated subquery references columns from the outer query.

```sql
-- Find employees who earn more than their department average
SELECT e1.FirstName, e1.LastName, e1.Department, e1.Salary
FROM Employees e1
WHERE e1.Salary > (
    SELECT AVG(e2.Salary) 
    FROM Employees e2 
    WHERE e2.Department = e1.Department  -- References outer query
);
```

**What happens:**
- For EACH employee in outer query:
  1. Inner query calculates average salary for that employee's department
  2. Compare employee's salary to department average

**Result:**
| FirstName | LastName | Department | Salary |
|-----------|----------|------------|--------|
| Sarah | Johnson | HR | 65000 |
| Mike | Davis | IT | 80000 |
| David | Brown | Sales | 72000 |
| Lisa | Anderson | IT | 78000 |
| Robert | Garcia | IT | 85000 |

---

## **5. EXISTS and NOT EXISTS**

### **EXISTS** - Check if subquery returns any rows

```sql
-- Find departments that have at least one employee in New York
SELECT DISTINCT Department
FROM Employees e1
WHERE EXISTS (
    SELECT 1 
    FROM Employees e2 
    WHERE e2.Department = e1.Department 
      AND e2.City = 'New York'
);
```

**Result:**
| Department |
|------------|
| IT |

---

### **NOT EXISTS** - Check if subquery returns NO rows

```sql
-- Find departments with NO employees in New York
SELECT DISTINCT Department
FROM Employees e1
WHERE NOT EXISTS (
    SELECT 1 
    FROM Employees e2 
    WHERE e2.Department = e1.Department 
      AND e2.City = 'New York'
);
```

**Result:**
| Department |
|------------|
| HR |
| Sales |
| Marketing |

---

## **6. Subquery with ALL**

```sql
-- Find employee(s) with highest salary
SELECT FirstName, LastName, Salary
FROM Employees 
WHERE Salary >= ALL (SELECT Salary FROM Employees);
```

**What happens:**
- `>= ALL` means greater than or equal to ALL values in subquery
- Finds maximum salary

**Result:**
| FirstName | LastName | Salary |
|-----------|----------|--------|
| Robert | Garcia | 85000 |

---

## **7. Subquery with ANY/SOME**

```sql
-- Find employees earning more than ANY IT employee
SELECT FirstName, LastName, Department, Salary
FROM Employees 
WHERE Salary > ANY (
    SELECT Salary 
    FROM Employees 
    WHERE Department = 'IT'
)
AND Department != 'IT';
```

**What happens:**
- `> ANY` means greater than at least one value
- IT salaries: 75000, 80000, 78000, 85000
- Finds non-IT employees earning more than at least one IT employee (>75000)

---

# CASE Statements

## **What is CASE?**
CASE is like an **if-then-else** statement in SQL. It returns different values based on conditions.

### **Syntax:**
```sql
CASE 
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE result3
END
```

---

## **1. Simple CASE**

```sql
SELECT 
    FirstName,
    LastName,
    Salary,
    CASE 
        WHEN Salary >= 80000 THEN 'High'
        WHEN Salary >= 70000 THEN 'Medium'
        ELSE 'Low'
    END AS SalaryLevel
FROM Employees;
```

**Result:**
| FirstName | LastName | Salary | SalaryLevel |
|-----------|----------|--------|-------------|
| John | Smith | 75000 | Medium |
| Sarah | Johnson | 65000 | Low |
| Mike | Davis | 80000 | High |
| Emily | Wilson | 70000 | Medium |
| David | Brown | 72000 | Medium |
| Lisa | Anderson | 78000 | Medium |
| James | Taylor | 62000 | Low |
| Anna | Martinez | 68000 | Low |
| Robert | Garcia | 85000 | High |
| Jennifer | Lee | 71000 | Medium |

---

## **2. CASE in SELECT with Multiple Conditions**

```sql
SELECT 
    FirstName,
    LastName,
    Department,
    Salary,
    CASE 
        WHEN Department = 'IT' AND Salary > 75000 THEN 'Senior IT'
        WHEN Department = 'IT' THEN 'Junior IT'
        WHEN Department = 'Sales' AND Salary > 70000 THEN 'Senior Sales'
        WHEN Department = 'Sales' THEN 'Junior Sales'
        ELSE Department
    END AS Position
FROM Employees;
```

**Result:**
| FirstName | LastName | Department | Salary | Position |
|-----------|----------|------------|--------|----------|
| John | Smith | IT | 75000 | Junior IT |
| Sarah | Johnson | HR | 65000 | HR |
| Mike | Davis | IT | 80000 | Senior IT |
| ... | ... | ... | ... | ... |

---

## **3. CASE with Aggregates**

```sql
SELECT 
    Department,
    COUNT(*) AS TotalEmployees,
    SUM(CASE WHEN Salary > 70000 THEN 1 ELSE 0 END) AS HighEarners,
    SUM(CASE WHEN Salary <= 70000 THEN 1 ELSE 0 END) AS LowEarners
FROM Employees 
GROUP BY Department;
```

**What happens:**
- Counts total employees per department
- Counts how many earn >70000
- Counts how many earn ≤70000

**Result:**
| Department | TotalEmployees | HighEarners | LowEarners |
|------------|----------------|-------------|------------|
| HR | 2 | 0 | 2 |
| IT | 4 | 4 | 0 |
| Marketing | 1 | 0 | 1 |
| Sales | 3 | 2 | 1 |

---

## **4. CASE in ORDER BY**

```sql
-- Custom sorting: IT first, then Sales, then others
SELECT FirstName, LastName, Department
FROM Employees 
ORDER BY 
    CASE Department
        WHEN 'IT' THEN 1
        WHEN 'Sales' THEN 2
        ELSE 3
    END,
    LastName;
```

**Result:** IT employees first, then Sales, then others, sorted by last name within each group

---

## **5. CASE in WHERE**

```sql
-- Flexible filtering based on condition
SELECT FirstName, LastName, Department, Salary
FROM Employees 
WHERE 
    CASE 
        WHEN Department = 'IT' THEN Salary > 75000
        WHEN Department = 'Sales' THEN Salary > 70000
        ELSE Salary > 65000
    END;
```

**What happens:**
- Different salary thresholds for different departments
- IT: salary must be >75000
- Sales: salary must be >70000
- Others: salary must be >65000

---

## **6. Simple CASE Syntax (Equality Only)**

```sql
SELECT 
    FirstName,
    LastName,
    Department,
    CASE Department
        WHEN 'IT' THEN 'Technology'
        WHEN 'HR' THEN 'Human Resources'
        WHEN 'Sales' THEN 'Sales & Marketing'
        ELSE 'Other'
    END AS DepartmentFullName
FROM Employees;
```

**Result:**
| FirstName | LastName | Department | DepartmentFullName |
|-----------|----------|------------|--------------------|
| John | Smith | IT | Technology |
| Sarah | Johnson | HR | Human Resources |
| ... | ... | ... | ... |

---

# String Functions

## **1. CONCAT - Combine Strings**

```sql
SELECT 
    CONCAT(FirstName, ' ', LastName) AS FullName,
    CONCAT(City, ', ', Country) AS Location
FROM Employees;
```

**Result:**
| FullName | Location |
|----------|----------|
| John Smith | New York, USA |
| Sarah Johnson | Los Angeles, USA |
| ... | ... |

---

## **2. UPPER and LOWER - Change Case**

```sql
SELECT 
    FirstName,
    UPPER(FirstName) AS Uppercase,
    LOWER(FirstName) AS Lowercase
FROM Employees 
LIMIT 3;
```

**Result:**
| FirstName | Uppercase | Lowercase |
|-----------|-----------|-----------|
| John | JOHN | john |
| Sarah | SARAH | sarah |
| Mike | MIKE | mike |

---

## **3. LENGTH - String Length**

```sql
SELECT 
    FirstName,
    LENGTH(FirstName) AS NameLength
FROM Employees;
```

**Result:**
| FirstName | NameLength |
|-----------|------------|
| John | 4 |
| Sarah | 5 |
| Mike | 4 |
| ... | ... |

---

## **4. SUBSTRING - Extract Part of String**

```sql
-- Extract first 3 characters
SELECT 
    Email,
    SUBSTRING(Email, 1, 3) AS FirstThree,
    SUBSTRING(Email, 1, POSITION('@' IN Email) - 1) AS Username
FROM Employees 
LIMIT 3;
```

**SUBSTRING(string, start_position, length)**

**Result:**
| Email | FirstThree | Username |
|-------|------------|----------|
| john.smith@company.com | joh | john.smith |
| sarah.j@company.com | sar | sarah.j |
| ... | ... | ... |

---

## **5. TRIM, LTRIM, RTRIM - Remove Spaces**

```sql
SELECT 
    TRIM('  Hello  ') AS Trimmed,          -- 'Hello'
    LTRIM('  Hello  ') AS LeftTrimmed,     -- 'Hello  '
    RTRIM('  Hello  ') AS RightTrimmed;    -- '  Hello'
```

---

## **6. REPLACE - Replace Text**

```sql
SELECT 
    Email,
    REPLACE(Email, '@company.com', '@newcompany.com') AS NewEmail
FROM Employees 
LIMIT 3;
```

**Result:**
| Email | NewEmail |
|-------|----------|
| john.smith@company.com | john.smith@newcompany.com |
| sarah.j@company.com | sarah.j@newcompany.com |
| ... | ... |

---

## **7. LEFT and RIGHT - Get Characters from Ends**

```sql
SELECT 
    FirstName,
    LEFT(FirstName, 3) AS First3,
    RIGHT(FirstName, 3) AS Last3
FROM Employees 
LIMIT 3;
```

**Result:**
| FirstName | First3 | Last3 |
|-----------|--------|-------|
| John | Joh | ohn |
| Sarah | Sar | rah |
| Mike | Mik | ike |

---

## **8. POSITION - Find Position of Substring**

```sql
SELECT 
    Email,
    POSITION('@' IN Email) AS AtPosition
FROM Employees 
LIMIT 3;
```

**Result:**
| Email | AtPosition |
|-------|------------|
| john.smith@company.com | 11 |
| sarah.j@company.com | 8 |
| ... | ... |

---

# Date and Time Functions

## **1. NOW() and CURDATE() - Current Date/Time**

```sql
SELECT 
    NOW() AS CurrentDateTime,      -- 2024-01-30 14:35:22
    CURDATE() AS CurrentDate,      -- 2024-01-30
    CURTIME() AS CurrentTime;      -- 14:35:22
```

---

## **2. DATE, MONTH, YEAR - Extract Parts**

```sql
SELECT 
    HireDate,
    YEAR(HireDate) AS HireYear,
    MONTH(HireDate) AS HireMonth,
    DAY(HireDate) AS HireDay,
    DAYNAME(HireDate) AS HireDayName
FROM Employees 
LIMIT 3;
```

**Result:**
| HireDate | HireYear | HireMonth | HireDay | HireDayName |
|----------|----------|-----------|---------|-------------|
| 2020-01-15 | 2020 | 1 | 15 | Wednesday |
| 2019-03-20 | 2019 | 3 | 20 | Wednesday |
| ... | ... | ... | ... | ... |

---

## **3. DATEDIFF - Difference Between Dates**

```sql
SELECT 
    FirstName,
    LastName,
    HireDate,
    DATEDIFF(CURDATE(), HireDate) AS DaysEmployed,
    ROUND(DATEDIFF(CURDATE(), HireDate) / 365, 1) AS YearsEmployed
FROM Employees;
```

**Result:**
| FirstName | LastName | HireDate | DaysEmployed | YearsEmployed |
|-----------|----------|----------|--------------|---------------|
| John | Smith | 2020-01-15 | 1476 | 4.0 |
| Sarah | Johnson | 2019-03-20 | 1777 | 4.9 |
| ... | ... | ... | ... | ... |

---

## **4. DATE_ADD and DATE_SUB - Add/Subtract Dates**

```sql
SELECT 
    FirstName,
    HireDate,
    DATE_ADD(HireDate, INTERVAL 1 YEAR) AS OneYearAnniversary,
    DATE_ADD(HireDate, INTERVAL 90 DAY) AS After90Days,
    DATE_SUB(CURDATE(), INTERVAL 1 YEAR) AS OneYearAgo
FROM Employees 
LIMIT 3;
```

---

## **5. TIMESTAMPDIFF - Precise Difference**

```sql
SELECT 
    FirstName,
    HireDate,
    TIMESTAMPDIFF(YEAR, HireDate, CURDATE()) AS YearsEmployed,
    TIMESTAMPDIFF(MONTH, HireDate, CURDATE()) AS MonthsEmployed,
    TIMESTAMPDIFF(DAY, HireDate, CURDATE()) AS DaysEmployed
FROM Employees 
LIMIT 3;
```

---

## **6. Filtering by Date**

```sql
-- Employees hired in 2020
SELECT FirstName, LastName, HireDate
FROM Employees 
WHERE YEAR(HireDate) = 2020;
```

```sql
-- Employees hired in last year
SELECT FirstName, LastName, HireDate
FROM Employees 
WHERE HireDate >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR);
```

```sql
-- Employees hired in Q1 (Jan-Mar)
SELECT FirstName, LastName, HireDate
FROM Employees 
WHERE MONTH(HireDate) BETWEEN 1 AND 3;
```

---

# NULL Handling

## **1. IS NULL and IS NOT NULL**

```sql
-- Employees without a manager (top level)
SELECT FirstName, LastName, ManagerID
FROM Employees 
WHERE ManagerID IS NULL;
```

**Result:**
| FirstName | LastName | ManagerID |
|-----------|----------|-----------|
| John | Smith | NULL |
| Emily | Wilson | NULL |
| Anna | Martinez | NULL |

---

## **2. COALESCE - Return First Non-NULL Value**

```sql
SELECT 
    FirstName,
    LastName,
    ManagerID,
    COALESCE(ManagerID, 0) AS ManagerIDOrZero,
    COALESCE(ManagerID, 'No Manager') AS ManagerIDOrText
FROM Employees 
LIMIT 5;
```

**Result:**
| FirstName | LastName | ManagerID | ManagerIDOrZero | ManagerIDOrText |
|-----------|----------|-----------|-----------------|-----------------|
| John | Smith | NULL | 0 | No Manager |
| Sarah | Johnson | 1 | 1 | 1 |
| ... | ... | ... | ... | ... |

---

## **3. IFNULL / NULLIF**

```sql
-- IFNULL(value, replacement) - MySQL specific
SELECT 
    FirstName,
    IFNULL(ManagerID, 0) AS ManagerIDOrZero
FROM Employees 
LIMIT 3;
```

```sql
-- NULLIF(value1, value2) - Returns NULL if equal
SELECT 
    NULLIF(5, 5),      -- Returns NULL
    NULLIF(5, 10);     -- Returns 5
```

---

## **4. Handling NULL in Calculations**

```sql
-- NULL in math operations returns NULL
SELECT 
    5 + NULL AS Result1,           -- NULL
    10 * NULL AS Result2,          -- NULL
    COALESCE(5 + NULL, 0) AS Result3;  -- 0
```

**Important:** 
- Any math operation with NULL returns NULL
- Use COALESCE to provide default values

---

# UNION, INTERSECT, EXCEPT

## **UNION - Combine Results**

### **What is UNION?**
Combines results from two or more SELECT statements. Removes duplicates.

```sql
-- Employees in IT or with salary > 75000
SELECT FirstName, LastName, 'IT Department' AS Reason
FROM Employees 
WHERE Department = 'IT'

UNION

SELECT FirstName, LastName, 'High Salary' AS Reason
FROM Employees 
WHERE Salary > 75000;
```

**Rules:**
- Same number of columns
- Compatible data types
- Column names from first query

---

## **UNION ALL - Include Duplicates**

```sql
-- Same as above but keeps duplicates
SELECT FirstName, LastName
FROM Employees 
WHERE Department = 'IT'

UNION ALL

SELECT FirstName, LastName
FROM Employees 
WHERE Salary > 75000;
```

**Difference:**
- **UNION** - Removes duplicates (slower)
- **UNION ALL** - Keeps duplicates (faster)

---

## **INTERSECT - Common Rows**

```sql
-- Employees in IT AND with salary > 75000
SELECT FirstName, LastName
FROM Employees 
WHERE Department = 'IT'

INTERSECT

SELECT FirstName, LastName
FROM Employees 
WHERE Salary > 75000;
```

**Note:** Not all databases support INTERSECT (alternative: use JOIN or WHERE with AND)

---

## **EXCEPT / MINUS - Difference**

```sql
-- Employees in IT but NOT with high salary
SELECT FirstName, LastName
FROM Employees 
WHERE Department = 'IT'

EXCEPT

SELECT FirstName, LastName
FROM Employees 
WHERE Salary > 80000;
```

**Note:** MySQL doesn't support EXCEPT (use LEFT JOIN with NULL check)

---

# Advanced Query Patterns

## **1. Top N Per Group**

```sql
-- Highest paid employee in each department
SELECT e1.*
FROM Employees e1
WHERE e1.Salary = (
    SELECT MAX(e2.Salary) 
    FROM Employees e2 
    WHERE e2.Department = e1.Department
);
```

---

## **2. Running Total**

```sql
SELECT 
    FirstName,
    LastName,
    Salary,
    (SELECT SUM(e2.Salary) 
     FROM Employees e2 
     WHERE e2.EmployeeID <= e1.EmployeeID) AS RunningTotal
FROM Employees e1
ORDER BY EmployeeID;
```

---

## **3. Rank Within Group**

```sql
-- Rank employees by salary within department
SELECT 
    FirstName,
    LastName,
    Department,
    Salary,
    (SELECT COUNT(*) + 1 
     FROM Employees e2 
     WHERE e2.Department = e1.Department 
       AND e2.Salary > e1.Salary) AS SalaryRank
FROM Employees e1
ORDER BY Department, SalaryRank;
```

---

## **4. Find Gaps in Sequences**

```sql
-- Find missing OrderIDs
SELECT o1.OrderID + 1 AS MissingID
FROM Orders o1
LEFT JOIN Orders o2 ON o1.OrderID + 1 = o2.OrderID
WHERE o2.OrderID IS NULL
  AND o1.OrderID < (SELECT MAX(OrderID) FROM Orders);
```

---

## **5. Pivot Data (Dynamic Columns)**

```sql
-- Count employees by department and salary range
SELECT 
    Department,
    SUM(CASE WHEN Salary < 70000 THEN 1 ELSE 0 END) AS Low,
    SUM(CASE WHEN Salary BETWEEN 70000 AND 80000 THEN 1 ELSE 0 END) AS Medium,
    SUM(CASE WHEN Salary > 80000 THEN 1 ELSE 0 END) AS High
FROM Employees 
GROUP BY Department;
```

---

# Query Execution Order

## **Logical Order SQL Executes Queries:**

```
1. FROM       - Get tables
2. JOIN       - Combine tables
3. WHERE      - Filter rows
4. GROUP BY   - Group rows
5. HAVING     - Filter groups
6. SELECT     - Choose columns
7. DISTINCT   - Remove duplicates
8. ORDER BY   - Sort results
9. LIMIT      - Limit rows
```

## **Example:**

```sql
SELECT Department, AVG(Salary) AS AvgSalary
FROM Employees 
WHERE HireDate > '2019-01-01'
GROUP BY Department
HAVING AVG(Salary) > 70000
ORDER BY AvgSalary DESC
LIMIT 3;
```

**Execution Steps:**
1. **FROM Employees** - Get Employees table
2. **WHERE HireDate > '2019-01-01'** - Filter to recent hires
3. **GROUP BY Department** - Group by department
4. **HAVING AVG(Salary) > 70000** - Keep groups with avg > 70000
5. **SELECT Department, AVG(Salary)** - Choose columns
6. **ORDER BY AvgSalary DESC** - Sort by average salary
7. **LIMIT 3** - Take top 3

---

# Complete Practice Examples

## **Example 1: Complex Employee Analysis**

```sql
SELECT 
    e.Department,
    COUNT(*) AS TotalEmployees,
    ROUND(AVG(e.Salary), 2) AS AvgSalary,
    MIN(e.Salary) AS MinSalary,
    MAX(e.Salary) AS MaxSalary,
    SUM(CASE WHEN e.HireDate >= DATE_SUB(CURDATE(), INTERVAL 2 YEAR) THEN 1 ELSE 0 END) AS RecentHires,
    CONCAT('$', FORMAT(SUM(e.Salary), 0)) AS TotalPayroll
FROM Employees e
WHERE e.Salary > 60000
GROUP BY e.Department
HAVING COUNT(*) >= 2
ORDER BY AvgSalary DESC;
```

---

## **Example 2: Product Sales Analysis**

```sql
SELECT 
    p.Category,
    COUNT(DISTINCT o.OrderID) AS TotalOrders,
    SUM(o.Quantity) AS TotalQuantitySold,
    ROUND(AVG(o.TotalAmount), 2) AS AvgOrderValue,
    SUM(o.TotalAmount) AS TotalRevenue,
    CASE 
        WHEN SUM(o.TotalAmount) > 2000 THEN 'High Performer'
        WHEN SUM(o.TotalAmount) > 1000 THEN 'Medium Performer'
        ELSE 'Low Performer'
    END AS PerformanceCategory
FROM Products p
INNER JOIN Orders o ON p.ProductID = o.ProductID
WHERE o.Status = 'Delivered'
GROUP BY p.Category
ORDER BY TotalRevenue DESC;
```

---

## **Example 3: Customer Behavior Analysis**

```sql
SELECT 
    o.CustomerName,
    COUNT(o.OrderID) AS TotalOrders,
    SUM(o.TotalAmount) AS TotalSpent,
    AVG(o.TotalAmount) AS AvgOrderValue,
    MIN(o.OrderDate) AS FirstOrder,
    MAX(o.OrderDate) AS LastOrder,
    DATEDIFF(MAX(o.OrderDate), MIN(o.OrderDate)) AS DaysBetweenFirstLast,
    CASE 
        WHEN COUNT(o.OrderID) >= 3 THEN 'Loyal Customer'
        WHEN COUNT(o.OrderID) = 2 THEN 'Repeat Customer'
        ELSE 'One-Time Customer'
    END AS CustomerType
FROM Orders o
GROUP BY o.CustomerName
HAVING COUNT(o.OrderID) >= 2
ORDER BY TotalSpent DESC;
```

---

## **Example 4: Inventory and Stock Analysis**

```sql
SELECT 
    p.ProductName,
    p.Category,
    p.Stock,
    COALESCE(SUM(o.Quantity), 0) AS TotalSold,
    p.Stock - COALESCE(SUM(o.Quantity), 0) AS RemainingStock,
    CASE 
        WHEN p.Stock - COALESCE(SUM(o.Quantity), 0) < 10 THEN 'Reorder Needed'
        WHEN p.Stock - COALESCE(SUM(o.Quantity), 0) < 50 THEN 'Low Stock'
        ELSE 'Adequate Stock'
    END AS StockStatus,
    ROUND((COALESCE(SUM(o.Quantity), 0) * 100.0 / p.Stock), 2) AS SoldPercentage
FROM Products p
LEFT JOIN Orders o ON p.ProductID = o.ProductID AND o.Status = 'Delivered'
GROUP BY p.ProductID, p.ProductName, p.Category, p.Stock
ORDER BY SoldPercentage DESC;
```

---

## **Example 5: Time-Based Trend Analysis**

```sql
SELECT 
    DATE_FORMAT(OrderDate, '%Y-%m') AS OrderMonth,
    COUNT(OrderID) AS OrderCount,
    SUM(TotalAmount) AS MonthlyRevenue,
    AVG(TotalAmount) AS AvgOrderValue,
    COUNT(DISTINCT CustomerName) AS UniqueCustomers,
    ROUND(SUM(TotalAmount) / COUNT(DISTINCT CustomerName), 2) AS RevenuePerCustomer
FROM Orders
WHERE OrderDate >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
GROUP BY DATE_FORMAT(OrderDate, '%Y-%m')
ORDER BY OrderMonth DESC;
```

---

## **Tips for Writing Queries:**

### **1. Start Simple, Build Up**
```sql
-- Step 1: Basic SELECT
SELECT * FROM Employees;

-- Step 2: Add specific columns
SELECT FirstName, LastName, Department FROM Employees;

-- Step 3: Add filtering
SELECT FirstName, LastName, Department FROM Employees WHERE Department = 'IT';

-- Step 4: Add grouping
SELECT Department, COUNT(*) FROM Employees GROUP BY Department;

-- Step 5: Add more aggregates
SELECT Department, COUNT(*), AVG(Salary) FROM Employees GROUP BY Department;
```

### **2. Test Each Part**
- Test WHERE clause alone
- Test GROUP BY without HAVING
- Add ORDER BY at the end
- Add LIMIT to see sample results

### **3. Use Aliases**
- Makes queries readable
- Essential for self-joins
- Required for subqueries in FROM

### **4. Comment Complex Queries**
```sql
-- Get department statistics for high-salary departments
SELECT 
    Department,
    COUNT(*) AS EmpCount,      -- Number of employees
    AVG(Salary) AS AvgSalary   -- Average salary
FROM Employees 
WHERE Salary > 60000           -- Only well-paid employees
GROUP BY Department
HAVING AVG(Salary) > 70000;   -- Only high-avg departments
```

---

## **Common Query Patterns to Remember:**

1. **Find duplicates:** `GROUP BY ... HAVING COUNT(*) > 1`
2. **Find missing:** `LEFT JOIN ... WHERE right_table.id IS NULL`
3. **Top N:** `ORDER BY ... LIMIT N`
4. **Percentage:** `(COUNT(*) * 100.0 / total)`
5. **Date ranges:** `WHERE date BETWEEN ... AND ...`
6. **Pattern matching:** `WHERE column LIKE '%pattern%'`
7. **Multiple conditions:** `WHERE (condition1 OR condition2) AND condition3`
8. **Subquery for filtering:** `WHERE column IN (SELECT ...)`

---

Happy querying! 🎯
