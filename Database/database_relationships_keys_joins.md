# Database Relationships, Keys, and Joins: Complete Guide

## Table of Contents
1. [Database Keys](#database-keys)
   - [Primary Key](#primary-key)
   - [Foreign Key](#foreign-key)
   - [Composite Key](#composite-key)
   - [Candidate Key](#candidate-key)
   - [Alternate Key](#alternate-key)
   - [Surrogate Key](#surrogate-key)
   - [Natural Key](#natural-key)
   - [Unique Key](#unique-key)
   - [Super Key](#super-key)
2. [Database Relationships](#database-relationships)
   - [One-to-One (1:1)](#one-to-one-11)
   - [One-to-Many (1:N)](#one-to-many-1n)
   - [Many-to-Many (M:N)](#many-to-many-mn)
   - [Self-Referencing](#self-referencing-relationship)
3. [SQL Joins](#sql-joins)
   - [INNER JOIN](#inner-join)
   - [LEFT JOIN (LEFT OUTER JOIN)](#left-join-left-outer-join)
   - [RIGHT JOIN (RIGHT OUTER JOIN)](#right-join-right-outer-join)
   - [FULL OUTER JOIN](#full-outer-join)
   - [CROSS JOIN](#cross-join)
   - [SELF JOIN](#self-join)
4. [Practical Examples](#practical-examples)
5. [Quick Reference](#quick-reference)
6. [Best Practices](#best-practices)

---

# Database Keys

Think of keys as **unique identifiers** or **labels** that help you find and organize data in your database. Like how your house has a unique address, or you have a unique social security number.

---

## Primary Key

### **What is it?**
A **Primary Key (PK)** is a column (or combination of columns) that **uniquely identifies each row** in a table. 

### **Rules:**
- ✅ Must be UNIQUE for every row
- ✅ Cannot be NULL (must have a value)
- ✅ Only ONE primary key per table
- ✅ Should never change once set

### **Think of it as:**
Your student ID card number - no two students have the same ID, and everyone must have one.

### **Example:**

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,          -- This is the Primary Key
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100)
);
```

**Sample Data:**

| StudentID (PK) | FirstName | LastName | Email |
|----------------|-----------|----------|-------|
| 1 | John | Smith | john@email.com |
| 2 | Sarah | Johnson | sarah@email.com |
| 3 | Mike | Davis | mike@email.com |

### **When to Use:**
- **ALWAYS** - Every table should have a primary key!
- When you need to uniquely identify each record
- When other tables need to reference this table

### **Common Patterns:**
- **Auto-increment ID:** `StudentID INT PRIMARY KEY AUTO_INCREMENT`
- **Natural identifier:** `ISBN VARCHAR(13) PRIMARY KEY` (for books)
- **UUID/GUID:** `UserID UUID PRIMARY KEY` (globally unique)

---

## Foreign Key

### **What is it?**
A **Foreign Key (FK)** is a column that **references the Primary Key of another table**. It creates a relationship between two tables.

### **Rules:**
- ✅ Must match a Primary Key value in the referenced table
- ✅ Can have duplicate values (many rows can reference the same PK)
- ✅ Can be NULL (if the relationship is optional)
- ✅ Enforces referential integrity

### **Think of it as:**
Your library card number that links you to the library's patron database. The card references YOU in another table.

### **Example:**

```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    OrderDate DATE,
    CustomerID INT,                     -- This is a Foreign Key
    TotalAmount DECIMAL(10, 2),
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,         -- Referenced by Orders table
    CustomerName VARCHAR(100),
    Email VARCHAR(100)
);
```

**Sample Data:**

**Customers Table:**
| CustomerID (PK) | CustomerName | Email |
|-----------------|--------------|-------|
| 1 | Alice Brown | alice@email.com |
| 2 | Bob Wilson | bob@email.com |

**Orders Table:**
| OrderID (PK) | OrderDate | CustomerID (FK) | TotalAmount |
|--------------|-----------|-----------------|-------------|
| 101 | 2024-01-15 | 1 | 150.00 |
| 102 | 2024-01-16 | 1 | 200.00 |
| 103 | 2024-01-17 | 2 | 75.50 |

**Notice:** 
- Alice (CustomerID = 1) has TWO orders
- The Foreign Key creates the link between tables

### **When to Use:**
- To create relationships between tables
- To maintain data integrity (can't have an order without a valid customer)
- To avoid data duplication

### **What Foreign Keys Prevent:**

```sql
-- ❌ This will FAIL - CustomerID 999 doesn't exist in Customers table
INSERT INTO Orders VALUES (104, '2024-01-18', 999, 50.00);

-- ❌ This will FAIL - Can't delete a customer who has orders
DELETE FROM Customers WHERE CustomerID = 1;

-- ✅ This works - Customer 1 exists
INSERT INTO Orders VALUES (104, '2024-01-18', 1, 50.00);
```

---

## Composite Key

### **What is it?**
A **Composite Key** is a Primary Key made up of **TWO OR MORE columns** together.

### **Think of it as:**
Your seat at a concert: Row + Seat Number together identify your unique seat. Row alone isn't unique, Seat Number alone isn't unique, but together they are!

### **Example:**

```sql
CREATE TABLE CourseEnrollments (
    StudentID INT,
    CourseID INT,
    Semester VARCHAR(20),
    Grade VARCHAR(2),
    PRIMARY KEY (StudentID, CourseID, Semester)  -- Composite Key
);
```

**Sample Data:**

| StudentID | CourseID | Semester | Grade |
|-----------|----------|----------|-------|
| 1 | 101 | Spring 2024 | A |
| 1 | 102 | Spring 2024 | B+ |
| 1 | 101 | Fall 2024 | A- |
| 2 | 101 | Spring 2024 | B |

**Why Composite?**
- Student 1 can enroll in Course 101 multiple times (different semesters)
- Student 1 can enroll in multiple courses in Spring 2024
- But the combination (Student 1, Course 101, Spring 2024) is UNIQUE

### **When to Use:**
- Junction tables (many-to-many relationships)
- When no single column uniquely identifies a row
- When the combination of attributes naturally forms uniqueness

### **Alternative Approach:**
Instead of a composite key, you can use a surrogate key:

```sql
CREATE TABLE CourseEnrollments (
    EnrollmentID INT PRIMARY KEY AUTO_INCREMENT,  -- Surrogate Key
    StudentID INT,
    CourseID INT,
    Semester VARCHAR(20),
    Grade VARCHAR(2),
    UNIQUE (StudentID, CourseID, Semester)        -- Still enforce uniqueness
);
```

---

## Candidate Key

### **What is it?**
A **Candidate Key** is any column (or combination of columns) that **COULD** be used as a Primary Key. They are "candidates" for being the primary key.

### **Think of it as:**
Ways to uniquely identify a person: Social Security Number, Passport Number, Driver's License Number. Each could be the primary key - they're all candidates!

### **Example:**

```sql
CREATE TABLE Employees (
    EmployeeID INT,           -- Candidate Key 1
    SSN VARCHAR(11),          -- Candidate Key 2
    Email VARCHAR(100),       -- Candidate Key 3
    EmployeeName VARCHAR(100),
    -- One of these will be chosen as PRIMARY KEY
    PRIMARY KEY (EmployeeID)
);
```

**All Candidate Keys:**
1. EmployeeID - unique for each employee
2. SSN - unique for each person
3. Email - unique in the company

**Only ONE becomes the Primary Key**, the others become Alternate Keys.

### **When to Identify:**
- During database design
- To understand all unique identifiers in your data
- To decide which one works best as the primary key

---

## Alternate Key

### **What is it?**
An **Alternate Key** is a Candidate Key that **was NOT chosen** as the Primary Key.

### **Think of it as:**
If you use your passport number as your primary ID, your driver's license number becomes an alternate way to identify you.

### **Example:**

```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,     -- This is the Primary Key
    SSN VARCHAR(11) UNIQUE,         -- Alternate Key
    Email VARCHAR(100) UNIQUE,      -- Alternate Key
    EmployeeName VARCHAR(100)
);
```

- **Primary Key:** EmployeeID
- **Alternate Keys:** SSN, Email

### **When to Use:**
- Declare them as UNIQUE constraints
- Still enforce uniqueness even if not the primary key
- Provide alternative ways to look up records

---

## Surrogate Key

### **What is it?**
A **Surrogate Key** is an **artificial key** created by the database (usually auto-increment) that has **no business meaning**.

### **Think of it as:**
An order number that's just generated sequentially (1, 2, 3...) rather than using something meaningful like customer name + date.

### **Example:**

```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY AUTO_INCREMENT,  -- Surrogate Key (no real-world meaning)
    ProductName VARCHAR(100),
    SKU VARCHAR(50),                           -- Natural Key (has meaning)
    Price DECIMAL(10, 2)
);
```

**Sample Data:**

| ProductID (Surrogate) | ProductName | SKU (Natural) | Price |
|-----------------------|-------------|---------------|-------|
| 1 | Laptop | TECH-LP-001 | 999.99 |
| 2 | Mouse | TECH-MS-002 | 29.99 |
| 3 | Keyboard | TECH-KB-003 | 79.99 |

### **Advantages:**
- ✅ Simple and short (just a number)
- ✅ Never changes (stable)
- ✅ No privacy concerns (meaningless number)
- ✅ Better performance (integers are fast)
- ✅ Easy to reference in other tables

### **When to Use:**
- **Almost always!** Modern databases favor surrogate keys
- When natural keys are long or composite
- When natural keys might change
- For better performance

---

## Natural Key

### **What is it?**
A **Natural Key** is a key that has **real-world meaning** and naturally exists in your data.

### **Think of it as:**
Using someone's email address or Social Security Number as their ID - it already exists and has meaning.

### **Example:**

```sql
CREATE TABLE Countries (
    CountryCode CHAR(2) PRIMARY KEY,  -- Natural Key (ISO country code)
    CountryName VARCHAR(100),
    Population INT
);
```

**Sample Data:**

| CountryCode (Natural PK) | CountryName | Population |
|--------------------------|-------------|------------|
| US | United States | 331000000 |
| UK | United Kingdom | 67000000 |
| CA | Canada | 38000000 |

### **Other Examples of Natural Keys:**
- ISBN for books
- VIN for vehicles
- Email addresses
- Social Security Numbers
- Product SKU codes

### **Disadvantages:**
- ❌ Can change (person changes email)
- ❌ Privacy concerns (SSN is sensitive)
- ❌ Sometimes long or complex
- ❌ Performance issues with strings vs integers

### **When to Use:**
- When the natural identifier is stable and guaranteed unique
- For lookup/reference tables (countries, currencies, states)
- When the natural key is simple and short

---

## Unique Key

### **What is it?**
A **Unique Key** constraint ensures that all values in a column are **different from each other**.

### **Difference from Primary Key:**
- ✅ Can have multiple UNIQUE constraints per table
- ✅ CAN allow NULL values (usually one NULL)
- ❌ Not automatically used for relationships

### **Example:**

```sql
CREATE TABLE Users (
    UserID INT PRIMARY KEY,
    Username VARCHAR(50) UNIQUE,      -- Must be unique
    Email VARCHAR(100) UNIQUE,        -- Must be unique
    PhoneNumber VARCHAR(15) UNIQUE    -- Must be unique
);
```

**Sample Data:**

| UserID (PK) | Username (UNIQUE) | Email (UNIQUE) | PhoneNumber (UNIQUE) |
|-------------|-------------------|----------------|----------------------|
| 1 | john_smith | john@email.com | 555-1234 |
| 2 | sarah_j | sarah@email.com | 555-5678 |
| 3 | mike_d | mike@email.com | 555-9012 |

**What Happens:**

```sql
-- ✅ This works
INSERT INTO Users VALUES (4, 'alice_w', 'alice@email.com', '555-1111');

-- ❌ This FAILS - Username 'john_smith' already exists
INSERT INTO Users VALUES (5, 'john_smith', 'newemail@email.com', '555-2222');

-- ❌ This FAILS - Email already exists
INSERT INTO Users VALUES (5, 'bob_m', 'john@email.com', '555-3333');
```

### **When to Use:**
- To enforce uniqueness on non-primary-key columns
- Email addresses, usernames, phone numbers
- Any column that should have unique values

---

## Super Key

### **What is it?**
A **Super Key** is any combination of columns that uniquely identifies a row. It might include extra, unnecessary columns.

### **Think of it as:**
Using your (Name + SSN + Birthday + Address) to identify you. It's unique, but SSN alone would work - the rest is extra!

### **Example:**

For this table:
| StudentID | Email | FirstName | LastName |
|-----------|-------|-----------|----------|
| 1 | john@email.com | John | Smith |

**Super Keys:**
- {StudentID} - Minimal (this is also a Candidate Key)
- {Email} - Minimal (this is also a Candidate Key)
- {StudentID, Email} - Has extra column
- {StudentID, FirstName, LastName} - Has extra columns
- {Email, FirstName} - Has extra column

**Candidate Keys are Super Keys with NO extra columns!**

### **When to Care:**
- Mostly a theoretical concept
- During database design to identify all possible unique combinations
- Not used directly in SQL

---

# Database Relationships

Relationships define **how tables are connected** to each other. Think of them as the "glue" that holds your database together!

---

## One-to-One (1:1)

### **What is it?**
One record in Table A is related to **exactly one** record in Table B, and vice versa.

### **Think of it as:**
- One person has one passport
- One car has one license plate
- One employee has one desk assignment

### **Visual Representation:**
```
Person ←──1:1──→ Passport
```

### **When to Use:**
1. **Sensitive Information Separation:** Keep sensitive data in a separate table
2. **Performance:** Split large tables with rarely-used columns
3. **Optional Attributes:** When some records have extra information others don't

### **Example 1: User Authentication**

```sql
-- Main user information
CREATE TABLE Users (
    UserID INT PRIMARY KEY AUTO_INCREMENT,
    Username VARCHAR(50) UNIQUE,
    Email VARCHAR(100),
    CreatedAt DATETIME
);

-- Sensitive authentication data (separate for security)
CREATE TABLE UserPasswords (
    UserID INT PRIMARY KEY,              -- Same as Users.UserID
    PasswordHash VARCHAR(255),
    LastPasswordChange DATETIME,
    FOREIGN KEY (UserID) REFERENCES Users(UserID)
);
```

**Sample Data:**

**Users Table:**
| UserID (PK) | Username | Email | CreatedAt |
|-------------|----------|-------|-----------|
| 1 | john_doe | john@email.com | 2024-01-01 |
| 2 | sarah_m | sarah@email.com | 2024-01-02 |

**UserPasswords Table:**
| UserID (PK, FK) | PasswordHash | LastPasswordChange |
|-----------------|--------------|-------------------|
| 1 | $2y$10$abc... | 2024-01-15 |
| 2 | $2y$10$xyz... | 2024-01-20 |

**Notice:** Each user has exactly ONE password record, and each password belongs to exactly ONE user.

### **Example 2: Employee Details**

```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Department VARCHAR(50)
);

CREATE TABLE EmployeeDetails (
    EmployeeID INT PRIMARY KEY,
    EmergencyContact VARCHAR(100),
    MedicalInfo TEXT,
    FOREIGN KEY (EmployeeID) REFERENCES Employees(EmployeeID)
);
```

**Why Separate?**
- Not all employees might have emergency contact info yet
- Medical info is sensitive and accessed rarely
- Keeps the main Employees table lean

### **Implementation Options:**

**Option 1: Foreign Key in Either Table**
```sql
-- FK in Table B pointing to Table A
CREATE TABLE TableB (
    ID INT PRIMARY KEY,
    TableA_ID INT UNIQUE,               -- UNIQUE ensures 1:1
    FOREIGN KEY (TableA_ID) REFERENCES TableA(ID)
);
```

**Option 2: Shared Primary Key** (Recommended)
```sql
-- Both tables share the same PK
CREATE TABLE TableB (
    ID INT PRIMARY KEY,                 -- Same as TableA's ID
    FOREIGN KEY (ID) REFERENCES TableA(ID)
);
```

---

## One-to-Many (1:N)

### **What is it?**
One record in Table A can be related to **many records** in Table B, but each record in Table B is related to only **one record** in Table A.

### **Think of it as:**
- One customer places many orders
- One teacher teaches many students
- One author writes many books
- One department has many employees

### **Visual Representation:**
```
Customer 1──→ many Orders
```

### **When to Use:**
- **Most common relationship type!**
- Parent-child relationships
- Categories and items
- One entity "owns" or "has" multiple of another

### **Example 1: Customers and Orders**

```sql
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY AUTO_INCREMENT,
    CustomerName VARCHAR(100),
    Email VARCHAR(100)
);

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY AUTO_INCREMENT,
    CustomerID INT,                      -- Foreign Key
    OrderDate DATE,
    TotalAmount DECIMAL(10, 2),
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);
```

**Sample Data:**

**Customers Table:**
| CustomerID (PK) | CustomerName | Email |
|-----------------|--------------|-------|
| 1 | Alice Brown | alice@email.com |
| 2 | Bob Wilson | bob@email.com |

**Orders Table:**
| OrderID (PK) | CustomerID (FK) | OrderDate | TotalAmount |
|--------------|-----------------|-----------|-------------|
| 101 | 1 | 2024-01-15 | 150.00 |
| 102 | 1 | 2024-01-20 | 200.00 |
| 103 | 1 | 2024-02-01 | 75.00 |
| 104 | 2 | 2024-01-18 | 300.00 |

**Notice:** 
- Customer 1 (Alice) has 3 orders → One-to-Many
- Customer 2 (Bob) has 1 order → Still One-to-Many (0 or more)
- Each order belongs to exactly ONE customer

### **Example 2: Blog Posts and Comments**

```sql
CREATE TABLE BlogPosts (
    PostID INT PRIMARY KEY AUTO_INCREMENT,
    Title VARCHAR(200),
    Content TEXT,
    AuthorID INT,
    PublishedDate DATETIME
);

CREATE TABLE Comments (
    CommentID INT PRIMARY KEY AUTO_INCREMENT,
    PostID INT,                          -- Foreign Key
    CommenterName VARCHAR(100),
    CommentText TEXT,
    CommentDate DATETIME,
    FOREIGN KEY (PostID) REFERENCES BlogPosts(PostID)
);
```

**Sample Data:**

**BlogPosts Table:**
| PostID (PK) | Title | AuthorID | PublishedDate |
|-------------|-------|----------|---------------|
| 1 | Introduction to SQL | 5 | 2024-01-10 |
| 2 | Advanced Joins | 5 | 2024-01-15 |

**Comments Table:**
| CommentID (PK) | PostID (FK) | CommenterName | CommentText | CommentDate |
|----------------|-------------|---------------|-------------|-------------|
| 1 | 1 | John | Great post! | 2024-01-11 |
| 2 | 1 | Sarah | Very helpful | 2024-01-11 |
| 3 | 1 | Mike | Thanks! | 2024-01-12 |
| 4 | 2 | Lisa | Excellent | 2024-01-16 |

**Notice:** Post 1 has 3 comments, Post 2 has 1 comment.

### **Key Point:**
**The Foreign Key ALWAYS goes in the "Many" side!**

---

## Many-to-Many (M:N)

### **What is it?**
Many records in Table A can relate to many records in Table B, and vice versa.

### **Think of it as:**
- Students enroll in many courses, courses have many students
- Books have many authors, authors write many books
- Movies have many actors, actors appear in many movies
- Products belong to many categories, categories have many products

### **Visual Representation:**
```
Students ←── many:many ──→ Courses
```

### **IMPORTANT:** You **cannot** directly implement many-to-many in SQL!

### **Solution: Junction Table (Associative Entity)**

A junction table sits between the two tables and breaks the M:N into TWO one-to-many relationships!

```
Students 1──→ many StudentCourses many ←──1 Courses
```

### **Example 1: Students and Courses**

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY AUTO_INCREMENT,
    StudentName VARCHAR(100),
    Email VARCHAR(100)
);

CREATE TABLE Courses (
    CourseID INT PRIMARY KEY AUTO_INCREMENT,
    CourseName VARCHAR(100),
    Credits INT
);

-- Junction Table (also called Bridge Table, Linking Table)
CREATE TABLE StudentCourses (
    StudentID INT,
    CourseID INT,
    EnrollmentDate DATE,
    Grade VARCHAR(2),
    PRIMARY KEY (StudentID, CourseID),   -- Composite Primary Key
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
);
```

**Sample Data:**

**Students Table:**
| StudentID (PK) | StudentName | Email |
|----------------|-------------|-------|
| 1 | Alice | alice@email.com |
| 2 | Bob | bob@email.com |
| 3 | Charlie | charlie@email.com |

**Courses Table:**
| CourseID (PK) | CourseName | Credits |
|---------------|------------|---------|
| 101 | Mathematics | 3 |
| 102 | Physics | 4 |
| 103 | Chemistry | 4 |

**StudentCourses Table (Junction):**
| StudentID (PK, FK) | CourseID (PK, FK) | EnrollmentDate | Grade |
|--------------------|-------------------|----------------|-------|
| 1 | 101 | 2024-01-15 | A |
| 1 | 102 | 2024-01-15 | B+ |
| 1 | 103 | 2024-01-15 | A- |
| 2 | 101 | 2024-01-16 | B |
| 2 | 102 | 2024-01-16 | B+ |
| 3 | 103 | 2024-01-17 | A |

**Notice:**
- Alice is enrolled in 3 courses (Math, Physics, Chemistry)
- Bob is enrolled in 2 courses (Math, Physics)
- Math course has 2 students (Alice, Bob)
- Chemistry has 2 students (Alice, Charlie)

### **Example 2: Books and Authors**

```sql
CREATE TABLE Authors (
    AuthorID INT PRIMARY KEY AUTO_INCREMENT,
    AuthorName VARCHAR(100),
    Country VARCHAR(50)
);

CREATE TABLE Books (
    BookID INT PRIMARY KEY AUTO_INCREMENT,
    BookTitle VARCHAR(200),
    ISBN VARCHAR(13),
    PublishedYear INT
);

-- Junction Table
CREATE TABLE BookAuthors (
    BookID INT,
    AuthorID INT,
    AuthorOrder INT,                     -- Additional info: 1st author, 2nd author, etc.
    PRIMARY KEY (BookID, AuthorID),
    FOREIGN KEY (BookID) REFERENCES Books(BookID),
    FOREIGN KEY (AuthorID) REFERENCES Authors(AuthorID)
);
```

**Sample Data:**

**Authors Table:**
| AuthorID (PK) | AuthorName | Country |
|---------------|------------|---------|
| 1 | J.K. Rowling | UK |
| 2 | Stephen King | USA |
| 3 | Peter Straub | USA |

**Books Table:**
| BookID (PK) | BookTitle | ISBN | PublishedYear |
|-------------|-----------|------|---------------|
| 1 | Harry Potter 1 | 9780439708180 | 1997 |
| 2 | Harry Potter 2 | 9780439064873 | 1998 |
| 3 | The Talisman | 9780345444882 | 1984 |
| 4 | The Shining | 9780307743657 | 1977 |

**BookAuthors Table (Junction):**
| BookID (FK) | AuthorID (FK) | AuthorOrder |
|-------------|---------------|-------------|
| 1 | 1 | 1 |
| 2 | 1 | 1 |
| 3 | 2 | 1 |
| 3 | 3 | 2 |
| 4 | 2 | 1 |

**Notice:**
- J.K. Rowling wrote 2 books
- Stephen King wrote 2 books (The Shining alone, The Talisman co-authored)
- The Talisman has 2 authors (King and Straub)

### **Junction Table Can Store Additional Information:**
- Enrollment date
- Grade
- Order/sequence
- Start/end dates
- Quantity
- Any relationship-specific data

---

## Self-Referencing Relationship

### **What is it?**
A table that has a relationship **with itself**. A record references another record in the same table.

### **Think of it as:**
- Employee → Manager (who is also an employee)
- Category → Parent Category
- Person → Spouse (both are people)
- Page → Parent Page (site hierarchy)

### **Visual Representation:**
```
Employee ──→ Manager (who is also an Employee)
```

### **Example 1: Employee-Manager Hierarchy**

```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY AUTO_INCREMENT,
    EmployeeName VARCHAR(100),
    JobTitle VARCHAR(50),
    ManagerID INT,                       -- Self-referencing Foreign Key
    FOREIGN KEY (ManagerID) REFERENCES Employees(EmployeeID)
);
```

**Sample Data:**

| EmployeeID (PK) | EmployeeName | JobTitle | ManagerID (FK) |
|-----------------|--------------|----------|----------------|
| 1 | Alice Johnson | CEO | NULL |
| 2 | Bob Smith | VP Sales | 1 |
| 3 | Carol White | VP Engineering | 1 |
| 4 | David Brown | Sales Manager | 2 |
| 5 | Eve Davis | Developer | 3 |
| 6 | Frank Wilson | Developer | 3 |

**Hierarchy:**
```
Alice (CEO)
├── Bob (VP Sales)
│   └── David (Sales Manager)
└── Carol (VP Engineering)
    ├── Eve (Developer)
    └── Frank (Developer)
```

**Notice:**
- Alice has no manager (CEO) → ManagerID = NULL
- Bob and Carol report to Alice (ManagerID = 1)
- David reports to Bob (ManagerID = 2)
- Eve and Frank report to Carol (ManagerID = 3)

### **Example 2: Category Hierarchy**

```sql
CREATE TABLE Categories (
    CategoryID INT PRIMARY KEY AUTO_INCREMENT,
    CategoryName VARCHAR(100),
    ParentCategoryID INT,                -- Self-referencing FK
    FOREIGN KEY (ParentCategoryID) REFERENCES Categories(CategoryID)
);
```

**Sample Data:**

| CategoryID (PK) | CategoryName | ParentCategoryID (FK) |
|-----------------|--------------|----------------------|
| 1 | Electronics | NULL |
| 2 | Computers | 1 |
| 3 | Laptops | 2 |
| 4 | Desktops | 2 |
| 5 | Phones | 1 |
| 6 | Smartphones | 5 |
| 7 | Feature Phones | 5 |

**Hierarchy:**
```
Electronics
├── Computers
│   ├── Laptops
│   └── Desktops
└── Phones
    ├── Smartphones
    └── Feature Phones
```

### **When to Use:**
- Organizational hierarchies
- Category/folder trees
- Comment threads (replies to comments)
- Network/graph structures
- Family trees

---

# SQL Joins

Joins are used to **combine data from multiple tables** based on related columns. Think of joins as bringing together information that's stored in different places!

---

## Visual Guide to Joins

Imagine two tables as circles (Venn diagram):

```
Table A          Table B
   ●●●●●●●     ●●●●●●●
  ●●●●●●●●●   ●●●●●●●●●
  ●●●●●●●●●●●●●●●●●●●●
  ●●●●●●●●●   ●●●●●●●●●
   ●●●●●●●     ●●●●●●●
```

Let's use these sample tables for all examples:

### **Sample Tables:**

**Customers Table:**
| CustomerID (PK) | CustomerName | City |
|-----------------|--------------|------|
| 1 | Alice | New York |
| 2 | Bob | Los Angeles |
| 3 | Charlie | Chicago |
| 4 | Diana | Houston |

**Orders Table:**
| OrderID (PK) | CustomerID (FK) | OrderDate | Amount |
|--------------|-----------------|-----------|--------|
| 101 | 1 | 2024-01-15 | 150.00 |
| 102 | 1 | 2024-01-20 | 200.00 |
| 103 | 2 | 2024-01-18 | 300.00 |
| 104 | 5 | 2024-01-25 | 100.00 |

**Notice:** 
- Customer 3 (Charlie) has no orders
- Customer 4 (Diana) has no orders
- Order 104 references CustomerID 5 (doesn't exist in Customers)

---

## INNER JOIN

### **What is it?**
Returns only the rows where there's a **match in BOTH tables**.

### **Visual:**
```
Table A          Table B
   ●●●●●●●     ●●●●●●●
  ●●●●●●●●●   ●●●●●●●●●
  ●●●●[XXXXX]●●●●●●●●●  ← Only this overlapping part
  ●●●●●●●●●   ●●●●●●●●●
   ●●●●●●●     ●●●●●●●
```

### **Syntax:**
```sql
SELECT columns
FROM Table1
INNER JOIN Table2
ON Table1.column = Table2.column;
```

### **Example:**

```sql
SELECT 
    Customers.CustomerName,
    Customers.City,
    Orders.OrderID,
    Orders.OrderDate,
    Orders.Amount
FROM Customers
INNER JOIN Orders
ON Customers.CustomerID = Orders.CustomerID;
```

### **Result:**

| CustomerName | City | OrderID | OrderDate | Amount |
|--------------|------|---------|-----------|--------|
| Alice | New York | 101 | 2024-01-15 | 150.00 |
| Alice | New York | 102 | 2024-01-20 | 200.00 |
| Bob | Los Angeles | 103 | 2024-01-18 | 300.00 |

**What Happened:**
- ✅ Alice appears twice (has 2 orders)
- ✅ Bob appears once (has 1 order)
- ❌ Charlie NOT included (no orders)
- ❌ Diana NOT included (no orders)
- ❌ Order 104 NOT included (CustomerID 5 doesn't exist in Customers)

### **When to Use:**
- When you want ONLY records that have matching data in both tables
- Finding customers who have placed orders
- Finding students enrolled in courses
- **Most common join type!**

### **Alternative Syntax (older style - still works):**
```sql
SELECT *
FROM Customers, Orders
WHERE Customers.CustomerID = Orders.CustomerID;
```

---

## LEFT JOIN (LEFT OUTER JOIN)

### **What is it?**
Returns **ALL rows from the LEFT table**, and matching rows from the right table. If no match, NULL values for right table columns.

### **Visual:**
```
Table A          Table B
   [●●●●●●●]    ●●●●●●●
  [●●●●●●●●●]  ●●●●●●●●●
  [●●●●XXXXX]●●●●●●●●●   ← All of left + overlap
  [●●●●●●●●●]  ●●●●●●●●●
   [●●●●●●●]    ●●●●●●●
```

### **Syntax:**
```sql
SELECT columns
FROM Table1
LEFT JOIN Table2
ON Table1.column = Table2.column;
```

### **Example:**

```sql
SELECT 
    Customers.CustomerName,
    Customers.City,
    Orders.OrderID,
    Orders.OrderDate,
    Orders.Amount
FROM Customers
LEFT JOIN Orders
ON Customers.CustomerID = Orders.CustomerID;
```

### **Result:**

| CustomerName | City | OrderID | OrderDate | Amount |
|--------------|------|---------|-----------|--------|
| Alice | New York | 101 | 2024-01-15 | 150.00 |
| Alice | New York | 102 | 2024-01-20 | 200.00 |
| Bob | Los Angeles | 103 | 2024-01-18 | 300.00 |
| Charlie | Chicago | NULL | NULL | NULL |
| Diana | Houston | NULL | NULL | NULL |

**What Happened:**
- ✅ Alice appears with both orders
- ✅ Bob appears with his order
- ✅ Charlie appears with NULL order data (no orders)
- ✅ Diana appears with NULL order data (no orders)
- ❌ Order 104 NOT included (CustomerID 5 not in Customers)

### **When to Use:**
- Find all customers (including those without orders)
- Find all students (including those not enrolled in courses)
- "Show me everyone, even if they don't have related data"
- **Very common for finding missing relationships!**

### **Find Customers Without Orders:**
```sql
SELECT Customers.CustomerName
FROM Customers
LEFT JOIN Orders
ON Customers.CustomerID = Orders.CustomerID
WHERE Orders.OrderID IS NULL;
```

**Result:**
| CustomerName |
|--------------|
| Charlie |
| Diana |

---

## RIGHT JOIN (RIGHT OUTER JOIN)

### **What is it?**
Returns **ALL rows from the RIGHT table**, and matching rows from the left table. If no match, NULL values for left table columns.

### **Visual:**
```
Table A          Table B
   ●●●●●●●     [●●●●●●●]
  ●●●●●●●●●   [●●●●●●●●●]
  ●●●●[XXXXX]●●●●●●●●●]  ← All of right + overlap
  ●●●●●●●●●   [●●●●●●●●●]
   ●●●●●●●     [●●●●●●●]
```

### **Syntax:**
```sql
SELECT columns
FROM Table1
RIGHT JOIN Table2
ON Table1.column = Table2.column;
```

### **Example:**

```sql
SELECT 
    Customers.CustomerName,
    Customers.City,
    Orders.OrderID,
    Orders.OrderDate,
    Orders.Amount
FROM Customers
RIGHT JOIN Orders
ON Customers.CustomerID = Orders.CustomerID;
```

### **Result:**

| CustomerName | City | OrderID | OrderDate | Amount |
|--------------|------|---------|-----------|--------|
| Alice | New York | 101 | 2024-01-15 | 150.00 |
| Alice | New York | 102 | 2024-01-20 | 200.00 |
| Bob | Los Angeles | 103 | 2024-01-18 | 300.00 |
| NULL | NULL | 104 | 2024-01-25 | 100.00 |

**What Happened:**
- ✅ All orders are included
- ✅ Matching customers shown with orders
- ✅ Order 104 appears with NULL customer data (CustomerID 5 doesn't exist)
- ❌ Charlie NOT included (no orders)
- ❌ Diana NOT included (no orders)

### **When to Use:**
- Less common than LEFT JOIN
- Find all orders (including orphaned ones without valid customers)
- Data quality checks (find invalid foreign keys)
- **Note:** You can usually rewrite RIGHT JOIN as LEFT JOIN by swapping table order!

### **RIGHT JOIN = LEFT JOIN with tables swapped:**
```sql
-- These are equivalent:
SELECT * FROM A RIGHT JOIN B ON A.id = B.id;
SELECT * FROM B LEFT JOIN A ON B.id = A.id;
```

---

## FULL OUTER JOIN

### **What is it?**
Returns **ALL rows from BOTH tables**. Where there's a match, combine them. Where no match, show NULL for the missing side.

### **Visual:**
```
Table A          Table B
   [●●●●●●●]    [●●●●●●●]
  [●●●●●●●●●]  [●●●●●●●●●]
  [●●●●XXXXX●●●●●●●●●]  ← Everything from both
  [●●●●●●●●●]  [●●●●●●●●●]
   [●●●●●●●]    [●●●●●●●]
```

### **Syntax:**
```sql
SELECT columns
FROM Table1
FULL OUTER JOIN Table2
ON Table1.column = Table2.column;
```

### **Example:**

```sql
SELECT 
    Customers.CustomerName,
    Customers.City,
    Orders.OrderID,
    Orders.OrderDate,
    Orders.Amount
FROM Customers
FULL OUTER JOIN Orders
ON Customers.CustomerID = Orders.CustomerID;
```

### **Result:**

| CustomerName | City | OrderID | OrderDate | Amount |
|--------------|------|---------|-----------|--------|
| Alice | New York | 101 | 2024-01-15 | 150.00 |
| Alice | New York | 102 | 2024-01-20 | 200.00 |
| Bob | Los Angeles | 103 | 2024-01-18 | 300.00 |
| Charlie | Chicago | NULL | NULL | NULL |
| Diana | Houston | NULL | NULL | NULL |
| NULL | NULL | 104 | 2024-01-25 | 100.00 |

**What Happened:**
- ✅ All customers (with or without orders)
- ✅ All orders (with or without valid customers)
- ✅ Matched records combined
- ✅ Unmatched records shown with NULLs

### **When to Use:**
- Data quality analysis (find mismatches on both sides)
- Complete picture of all records
- Migration/merge scenarios
- **Note:** Not all databases support FULL OUTER JOIN (MySQL doesn't!)

### **MySQL Alternative (simulate FULL OUTER JOIN):**
```sql
SELECT * FROM Customers LEFT JOIN Orders ON Customers.CustomerID = Orders.CustomerID
UNION
SELECT * FROM Customers RIGHT JOIN Orders ON Customers.CustomerID = Orders.CustomerID;
```

---

## CROSS JOIN

### **What is it?**
Returns the **Cartesian product** - every row from Table A combined with every row from Table B.

### **Visual:**
```
Table A (3 rows)  ×  Table B (4 rows)  =  12 result rows
```

### **Syntax:**
```sql
SELECT columns
FROM Table1
CROSS JOIN Table2;
```

### **Example:**

**Sizes Table:**
| SizeID | SizeName |
|--------|----------|
| 1 | Small |
| 2 | Medium |
| 3 | Large |

**Colors Table:**
| ColorID | ColorName |
|---------|-----------|
| 1 | Red |
| 2 | Blue |
| 3 | Green |

```sql
SELECT 
    Sizes.SizeName,
    Colors.ColorName
FROM Sizes
CROSS JOIN Colors;
```

### **Result (9 rows = 3 × 3):**

| SizeName | ColorName |
|----------|-----------|
| Small | Red |
| Small | Blue |
| Small | Green |
| Medium | Red |
| Medium | Blue |
| Medium | Green |
| Large | Red |
| Large | Blue |
| Large | Green |

**What Happened:**
- Every size combined with every color
- 3 sizes × 3 colors = 9 combinations

### **When to Use:**
- Generate all possible combinations
- Product variants (size × color × material)
- Scheduling (employees × time slots)
- Test data generation
- **Warning:** Can create HUGE result sets! (1000 × 1000 = 1,000,000 rows)

### **Alternative Syntax:**
```sql
-- These are equivalent:
SELECT * FROM A CROSS JOIN B;
SELECT * FROM A, B;
```

---

## SELF JOIN

### **What is it?**
A table joined with **itself**. Used to compare rows within the same table.

### **Syntax:**
```sql
SELECT columns
FROM Table1 AS alias1
JOIN Table1 AS alias2
ON alias1.column = alias2.column;
```

### **Example 1: Employee-Manager Relationships**

**Employees Table:**
| EmployeeID | EmployeeName | ManagerID |
|------------|--------------|-----------|
| 1 | Alice | NULL |
| 2 | Bob | 1 |
| 3 | Charlie | 1 |
| 4 | David | 2 |
| 5 | Eve | 2 |

```sql
SELECT 
    e.EmployeeName AS Employee,
    m.EmployeeName AS Manager
FROM Employees AS e
LEFT JOIN Employees AS m
ON e.ManagerID = m.EmployeeID;
```

### **Result:**

| Employee | Manager |
|----------|---------|
| Alice | NULL |
| Bob | Alice |
| Charlie | Alice |
| David | Bob |
| Eve | Bob |

**What Happened:**
- Each employee matched with their manager (who is also in the same table)
- Alice has no manager (CEO)
- Bob and Charlie report to Alice
- David and Eve report to Bob

### **Example 2: Find Employees in Same Department**

**Employees Table:**
| EmployeeID | EmployeeName | Department |
|------------|--------------|------------|
| 1 | Alice | Sales |
| 2 | Bob | Sales |
| 3 | Charlie | IT |
| 4 | David | IT |

```sql
SELECT 
    e1.EmployeeName AS Employee1,
    e2.EmployeeName AS Employee2,
    e1.Department
FROM Employees AS e1
JOIN Employees AS e2
ON e1.Department = e2.Department
AND e1.EmployeeID < e2.EmployeeID;  -- Avoid duplicates
```

### **Result:**

| Employee1 | Employee2 | Department |
|-----------|-----------|------------|
| Alice | Bob | Sales |
| Charlie | David | IT |

**What Happened:**
- Found pairs of employees in the same department
- `e1.EmployeeID < e2.EmployeeID` prevents showing (Alice, Bob) and (Bob, Alice)

### **When to Use:**
- Hierarchical data (managers, categories)
- Finding relationships within same entity
- Comparing records to each other
- Building trees or graphs

---

# Practical Examples

## Example 1: E-commerce Database

### **Complete Schema:**

```sql
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY AUTO_INCREMENT,
    CustomerName VARCHAR(100),
    Email VARCHAR(100) UNIQUE,
    City VARCHAR(50)
);

CREATE TABLE Products (
    ProductID INT PRIMARY KEY AUTO_INCREMENT,
    ProductName VARCHAR(100),
    Price DECIMAL(10, 2),
    Stock INT
);

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY AUTO_INCREMENT,
    CustomerID INT,
    OrderDate DATE,
    TotalAmount DECIMAL(10, 2),
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

CREATE TABLE OrderItems (
    OrderItemID INT PRIMARY KEY AUTO_INCREMENT,
    OrderID INT,
    ProductID INT,
    Quantity INT,
    Price DECIMAL(10, 2),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);
```

### **Relationships:**
- Customers → Orders (1:Many)
- Orders → OrderItems (1:Many)
- Products → OrderItems (1:Many)
- Orders ↔ Products (Many:Many through OrderItems)

### **Complex Query: Customer Order Details**

```sql
SELECT 
    c.CustomerName,
    c.Email,
    o.OrderID,
    o.OrderDate,
    p.ProductName,
    oi.Quantity,
    oi.Price,
    (oi.Quantity * oi.Price) AS LineTotal
FROM Customers c
INNER JOIN Orders o ON c.CustomerID = o.CustomerID
INNER JOIN OrderItems oi ON o.OrderID = oi.OrderID
INNER JOIN Products p ON oi.ProductID = p.ProductID
ORDER BY o.OrderDate DESC, o.OrderID;
```

**This shows:**
- Customer information
- Their orders
- Products in each order
- Quantities and prices
- Calculated line totals

---

## Example 2: School Database

### **Schema:**

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    StudentName VARCHAR(100),
    Email VARCHAR(100)
);

CREATE TABLE Courses (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(100),
    Credits INT
);

CREATE TABLE Instructors (
    InstructorID INT PRIMARY KEY,
    InstructorName VARCHAR(100),
    Department VARCHAR(50)
);

CREATE TABLE CourseInstructors (
    CourseID INT,
    InstructorID INT,
    Semester VARCHAR(20),
    PRIMARY KEY (CourseID, InstructorID, Semester),
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID),
    FOREIGN KEY (InstructorID) REFERENCES Instructors(InstructorID)
);

CREATE TABLE Enrollments (
    StudentID INT,
    CourseID INT,
    Semester VARCHAR(20),
    Grade VARCHAR(2),
    PRIMARY KEY (StudentID, CourseID, Semester),
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
);
```

### **Useful Queries:**

**1. Students with their courses and instructors:**
```sql
SELECT 
    s.StudentName,
    c.CourseName,
    i.InstructorName,
    e.Grade,
    e.Semester
FROM Students s
INNER JOIN Enrollments e ON s.StudentID = e.StudentID
INNER JOIN Courses c ON e.CourseID = c.CourseID
INNER JOIN CourseInstructors ci ON c.CourseID = ci.CourseID AND e.Semester = ci.Semester
INNER JOIN Instructors i ON ci.InstructorID = i.InstructorID;
```

**2. Find students not enrolled in any courses:**
```sql
SELECT s.StudentName, s.Email
FROM Students s
LEFT JOIN Enrollments e ON s.StudentID = e.StudentID
WHERE e.StudentID IS NULL;
```

**3. Courses with no students enrolled:**
```sql
SELECT c.CourseName, c.Credits
FROM Courses c
LEFT JOIN Enrollments e ON c.CourseID = e.CourseID
WHERE e.CourseID IS NULL;
```

---

# Quick Reference

## Join Types Summary

| Join Type | Returns | Use Case |
|-----------|---------|----------|
| **INNER JOIN** | Only matching rows from both tables | Find related records |
| **LEFT JOIN** | All from left + matching from right | Find all including those without matches |
| **RIGHT JOIN** | All from right + matching from left | Same as LEFT (less common) |
| **FULL OUTER JOIN** | All rows from both tables | Complete picture, data quality |
| **CROSS JOIN** | Cartesian product (all combinations) | Generate combinations |
| **SELF JOIN** | Table joined with itself | Hierarchies, comparisons |

## Key Types Summary

| Key Type | Description | Rules |
|----------|-------------|-------|
| **Primary Key** | Unique identifier for each row | Must be unique, NOT NULL, one per table |
| **Foreign Key** | References Primary Key in another table | Creates relationships, can be NULL |
| **Composite Key** | Multiple columns together form key | Combination must be unique |
| **Candidate Key** | Could be a Primary Key | Any column(s) that uniquely identify rows |
| **Alternate Key** | Candidate Key not chosen as PK | Enforce with UNIQUE constraint |
| **Surrogate Key** | Artificial key (auto-increment) | No business meaning, best for performance |
| **Natural Key** | Real-world identifier | Has business meaning (SSN, email) |
| **Unique Key** | Must be unique (not PK) | Can have multiple per table, can be NULL |

## Relationship Types Summary

| Relationship | Description | Implementation |
|--------------|-------------|----------------|
| **One-to-One** | One record ↔ One record | FK in either table with UNIQUE constraint |
| **One-to-Many** | One record → Many records | FK in the "many" table |
| **Many-to-Many** | Many ↔ Many | Junction table with two FKs |
| **Self-Referencing** | Table references itself | FK references same table's PK |

---

# Best Practices

## Database Design Best Practices

### **1. Primary Keys:**
✅ **DO:**
- Always have a primary key in every table
- Use surrogate keys (auto-increment IDs) for most tables
- Keep primary keys short and simple
- Never reuse primary key values

❌ **DON'T:**
- Use business data as primary keys if it might change
- Use multi-column composite keys unless necessary
- Allow primary keys to be NULL

### **2. Foreign Keys:**
✅ **DO:**
- Always define foreign key constraints
- Index foreign key columns for performance
- Use ON DELETE and ON UPDATE clauses appropriately
- Name foreign keys clearly (e.g., `FK_Orders_CustomerID`)

```sql
FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
    ON DELETE CASCADE          -- Delete orders when customer deleted
    ON UPDATE CASCADE          -- Update orders when customer ID changes
```

❌ **DON'T:**
- Create orphaned records (orders without customers)
- Skip foreign key constraints for "performance" (bad idea!)

### **3. Relationships:**
✅ **DO:**
- Draw ER diagrams before coding
- Identify cardinality (1:1, 1:N, M:N)
- Use junction tables for many-to-many
- Name junction tables clearly (`StudentCourses`, `BookAuthors`)

❌ **DON'T:**
- Try to implement many-to-many directly
- Create circular dependencies
- Over-normalize (sometimes denormalization is OK)

### **4. Joins:**
✅ **DO:**
- Use INNER JOIN when you want only matching records
- Use LEFT JOIN to include all records from main table
- Always specify join conditions in ON clause
- Use table aliases for readability

```sql
-- Good: Clear aliases
SELECT c.CustomerName, o.OrderID
FROM Customers c
INNER JOIN Orders o ON c.CustomerID = o.CustomerID;

-- Bad: No aliases, hard to read
SELECT Customers.CustomerName, Orders.OrderID
FROM Customers
INNER JOIN Orders ON Customers.CustomerID = Orders.CustomerID;
```

❌ **DON'T:**
- Use old-style joins (comma syntax)
- Forget WHERE clause when doing CROSS JOIN
- Join on non-indexed columns (slow!)

### **5. Naming Conventions:**
✅ **DO:**
- Use singular table names (`Customer` not `Customers`) OR consistent plural
- Use PascalCase or snake_case consistently
- Name foreign keys clearly (`CustomerID`, not just `ID`)
- Prefix constraint names (`PK_`, `FK_`, `UQ_`)

❌ **DON'T:**
- Mix naming styles
- Use reserved words as names
- Use special characters or spaces

---

## Performance Tips

### **1. Indexing:**
```sql
-- Create indexes on foreign keys
CREATE INDEX idx_orders_customer ON Orders(CustomerID);

-- Create indexes on frequently joined columns
CREATE INDEX idx_products_category ON Products(CategoryID);

-- Composite index for multiple columns
CREATE INDEX idx_orders_customer_date ON Orders(CustomerID, OrderDate);
```

### **2. Query Optimization:**
- Select only needed columns (avoid `SELECT *`)
- Use INNER JOIN instead of LEFT JOIN when possible
- Filter early with WHERE clauses
- Use LIMIT to restrict results
- Avoid functions on indexed columns in WHERE

```sql
-- Slow: Function on indexed column
SELECT * FROM Orders WHERE YEAR(OrderDate) = 2024;

-- Fast: Direct comparison
SELECT * FROM Orders WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01';
```

### **3. Avoid N+1 Queries:**
```sql
-- Bad: Separate query for each customer's orders
SELECT * FROM Customers;
-- Then for each customer:
SELECT * FROM Orders WHERE CustomerID = ?;

-- Good: Single JOIN query
SELECT c.*, o.*
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID;
```

---

## Common Mistakes to Avoid

### **Mistake 1: No Foreign Key Constraints**
```sql
-- ❌ Bad: No constraint
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT  -- Can be any value!
);

-- ✅ Good: With constraint
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);
```

### **Mistake 2: Wrong JOIN Type**
```sql
-- ❌ Bad: Using INNER JOIN when you need all customers
SELECT c.CustomerName, COUNT(o.OrderID) AS OrderCount
FROM Customers c
INNER JOIN Orders o ON c.CustomerID = o.CustomerID  -- Excludes customers with no orders
GROUP BY c.CustomerName;

-- ✅ Good: Using LEFT JOIN
SELECT c.CustomerName, COUNT(o.OrderID) AS OrderCount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID  -- Includes all customers
GROUP BY c.CustomerName;
```

### **Mistake 3: Missing Junction Table**
```sql
-- ❌ Bad: Trying to store multiple authors in one column
CREATE TABLE Books (
    BookID INT PRIMARY KEY,
    Title VARCHAR(200),
    Authors VARCHAR(500)  -- "King, Straub" - NO!
);

-- ✅ Good: Junction table
CREATE TABLE Books (
    BookID INT PRIMARY KEY,
    Title VARCHAR(200)
);

CREATE TABLE Authors (
    AuthorID INT PRIMARY KEY,
    AuthorName VARCHAR(100)
);

CREATE TABLE BookAuthors (
    BookID INT,
    AuthorID INT,
    PRIMARY KEY (BookID, AuthorID),
    FOREIGN KEY (BookID) REFERENCES Books(BookID),
    FOREIGN KEY (AuthorID) REFERENCES Authors(AuthorID)
);
```

---

## Conclusion

**Remember the Core Concepts:**

1. **Keys** identify and relate records
2. **Relationships** connect tables logically
3. **Joins** combine data from multiple tables

**The Foundation:**
- Every table needs a PRIMARY KEY
- Use FOREIGN KEYS to create relationships
- Choose the right relationship type (1:1, 1:N, M:N)
- Use the appropriate JOIN for your query needs

**Start Simple:**
- Begin with the entities you need (Students, Courses, Orders)
- Identify how they relate (Students enroll in Courses)
- Create junction tables for many-to-many
- Use INNER JOIN for most queries, LEFT JOIN when you need all records

**Practice Makes Perfect:**
- Draw ER diagrams
- Write queries on sample data
- Test different JOIN types
- Analyze query performance

Happy database designing! 🗄️
