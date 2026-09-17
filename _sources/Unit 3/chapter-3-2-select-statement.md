# Retrieving Data and Simple Sorting (The SELECT Statement)

#### Introduction: Asking Questions with the SELECT Statement
Now that you understand the broad capabilities of SQL and the seven fundamental pillars of data retrieval, it is time to write your very first queries. Throughout this unit, we will use Microsoft's sample database, **Northwind Traders**—a fictitious national distributor of gourmet foods—to practice writing SQL code in a realistic business setting.

In computer programming, the most basic task is extracting data from a table. In SQL, this is accomplished using the **SELECT** statement. 

At its simplest, a query requires two clauses:
1. **`SELECT`**: Specifies which columns you want to view.
2. **`FROM`**: Specifies the database table containing the data.

Consider the simplest possible SQL query:

```sql
SELECT *
FROM Employees;
```

In SQL, the asterisk (`*`) is a shortcut wildcard that means "retrieve all columns." When you execute this query, the database returns every row and every column from the `Employees` table.

```
+------------+----------+-----------+---------------------+-----------------+...
| EmployeeID | LastName | FirstName | Title               | TitleOfCourtesy |...
+------------+----------+-----------+---------------------+-----------------+...
| 1          | Davolio  | Nancy     | Sales Representative| Ms.             |...
| 2          | Fuller   | Andrew    | Vice President, Sales| Dr.             |...
| 3          | Leverling| Janet     | Sales Representative| Ms.             |...
+------------+----------+-----------+---------------------+-----------------+...
```

```{note}
**Data Integrity Rule:** Executing a `SELECT` query **never alters or modifies** the data stored in a database. A `SELECT` statement simply returns a temporary, read-only copy of the requested data for you to view or export.
```

---

#### Selecting Specific Columns and Code Style

While using `SELECT *` is convenient when exploring a table for the first time, retrieving every column is rarely necessary in professional reporting. Large enterprise tables can contain hundreds of columns. Pulling unnecessary fields wastes memory and clutters your analysis.

##### Explicit Column Selection
To retrieve specific columns, list their names after the `SELECT` keyword, separated by commas:

```sql
SELECT EmployeeID, FirstName, LastName, Title
FROM Employees;
```

You can select columns in **any order you choose**, regardless of how they are physically arranged in the underlying database table. You can even request the same column multiple times if needed.

##### SQL Syntax and Formatting Rules
As you write SQL queries, keep these fundamental syntax rules in mind:

* **SQL is Not Case-Sensitive:** SQL keywords (`SELECT`, `FROM`) and column names can be written in uppercase, lowercase, or mixed case. The queries `select firstname from employees;` and `SELECT FirstName FROM Employees;` will execute identically. However, by convention, we **capitalize SQL keywords** and use **CamelCase** for column names to make our code clean and readable.
* **Flexible Spacing and Line Breaks:** SQL ignores extra spaces, tabs, and line breaks. You can write an entire query on a single line or spread it across multiple lines.
* **No Spaces in Identifiers:** You cannot insert spaces inside a column or table name. For example, `LastName` is a valid identifier, whereas `Last Name` will cause a syntax error unless enclosed in square brackets (`[Last Name]`).
* **Comma Placement:** Column names in a `SELECT` clause must be separated by commas, but there must be **no comma after the final column** before the `FROM` keyword.

Here is an example demonstrating how spacing and indentation improve code readability:

```sql
-- Clean, professional formatting
SELECT 
    EmployeeID, 
    FirstName, 
    LastName, 
    Title
FROM Employees;
```

Writing clean, well-formatted code is not just about aesthetics—it is a core accounting control that makes your logic easy to audit, review, and maintain.

---

#### Thinking Like an Analyst: Four Ways to Answer "How Many Employees?"

To see how analytical thinking precedes code, let us tackle a simple business question: **How many employees work at Northwind Traders?**

In a spreadsheet, you might scroll to the bottom of a worksheet or highlight a column to check the status bar. In SQL, we must express our logical process explicitly. There are four distinct ways to answer this question using SQL, each with its own underlying logic and operational risks.

##### Method 1: Retrieve All Rows (`SELECT *`)
The most straightforward approach is to retrieve all rows from the `Employees` table and inspect the total record count reported by your database tool (such as Visual Studio Code or DBeaver):

```sql
SELECT *
FROM Employees;
```

When VS Code executes this query, the query results window displays all records and notes at the bottom: *"9 rows returned."* 

While this works for small tables, retrieving millions of rows just to read a count from a software status bar is inefficient and slow.

##### Method 2a: Multi-Step Logic (`TOP` + `ORDER BY`)
If we know that every employee is assigned a unique, sequential `EmployeeID` starting at 1, we can sort the table in descending order by `EmployeeID` and retrieve the single highest ID:

```sql
SELECT TOP (1) EmployeeID
FROM Employees
ORDER BY EmployeeID DESC;
```

This query sorts the table so the largest `EmployeeID` appears at the top, and `TOP (1)` restricts the output to that first row, returning `9`.

##### Method 2b: Aggregate Function (`MAX`)
Rather than sorting the entire table to find the top row, we can use SQL's built-in `MAX()` function to find the highest value in the `EmployeeID` column directly:

```sql
SELECT MAX(EmployeeID)
FROM Employees;
```

This query returns a single summary result: `9`.

##### Method 2c: Direct Row Counting (`COUNT`)
The most direct way to count records in SQL is using the `COUNT()` function:

```sql
SELECT COUNT(*)
FROM Employees;
```

This query instructs the database to count all rows in the `Employees` table, returning `9`.

```{warning}
**The Auditor's Perspective: Evaluating Operational Risks**
Why are Methods 2a and 2b risky in professional practice?

Methods 2a (`TOP (1)` with `ORDER BY DESC`) and 2b (`MAX`) rely on a dangerous assumption: that `EmployeeID` values are strictly sequential without gaps. Imagine an employee was hired (assigned ID 5) and later terminated, and their record was deleted from the database. Or imagine an enterprise system where primary key IDs are generated in non-sequential blocks. In these scenarios, the maximum `EmployeeID` might be `9`, but there may only be `8` active employees in the table!

Methods 2a and 2b report the **highest ID number**, not the **actual headcount**. Method 2c (`COUNT(*)`) is the only logically sound approach because it counts actual data records rather than inferring counts from ID sequence numbers.
```

---

#### Sorting Results with ORDER BY

By default, a database does not guarantee any specific order when returning query results. To sort your data, you must explicitly add an **`ORDER BY`** clause.

The `ORDER BY` clause syntax is:

```sql
SELECT Column1, Column2
FROM TableName
ORDER BY ColumnToSort [ASC | DESC];
```

* **Ascending Order (`ASC`):** Sorts numbers from smallest to largest, dates from oldest to newest, and text alphabetically (A to Z). This is the default setting in SQL. You can include the `ASC` keyword, but it is optional.
* **Descending Order (`DESC`):** Sorts numbers from largest to smallest, dates from newest to oldest, and text in reverse alphabetical order (Z to A). You must explicitly append `DESC` after the column name.

##### Single-Column Sort Example
Imagine the sales manager wants a list of all Northwind employees, sorted from youngest to oldest based on their birth date:

```sql
SELECT EmployeeID, FirstName, LastName, BirthDate
FROM Employees
ORDER BY BirthDate DESC;
```

Because we specified `DESC`, the employee with the most recent birth date appears first.

```
+------------+-----------+----------+------------+
| EmployeeID | FirstName | LastName | BirthDate  |
+------------+-----------+----------+------------+
| 9          | Anne      | Dodsworth| 1966-01-27 |
| 6          | Michael   | Suyama   | 1963-07-02 |
| 1          | Nancy     | Davolio  | 1948-12-08 |
+------------+-----------+----------+------------+
```

##### Clause Sequence Rule
In SQL, clause order is strictly enforced. The `ORDER BY` clause must always appear **after** the `FROM` clause:

1. `SELECT`
2. `FROM`
3. `ORDER BY`

Attempting to place `ORDER BY` before `FROM` will result in a syntax error.

---

#### Limiting Output with TOP

In large corporate databases, tables often contain millions of transaction rows. Running a query that returns millions of rows can freeze your computer and strain network bandwidth. When exploring data, you will often want to view only a small sample of records using the **`TOP`** clause.

##### The `TOP` Clause Syntax
The `TOP` clause specifies the maximum number (or percentage) of rows to return:

```sql
SELECT TOP (n) ColumnList
FROM TableName;
```

For example, to retrieve the first 5 records from the `Orders` table:

```sql
SELECT TOP (5) OrderID, CustomerID, OrderDate
FROM Orders;
```

You can also specify a percentage of rows using `TOP (n) PERCENT`:

```sql
SELECT TOP (10) PERCENT OrderID, CustomerID, OrderDate
FROM Orders;
```

##### Pairing `TOP` with `ORDER BY`
```{important}
Without an `ORDER BY` clause, using `TOP` returns an arbitrary set of rows based on whatever order the database happens to store them in on disk. To obtain meaningful, deterministic results—such as the top 5 highest sales transactions or the 3 most recent orders—you **must pair `TOP` with `ORDER BY`**.
```

Consider finding the three most expensive products sold by Northwind Traders:

```sql
SELECT TOP (3) ProductID, ProductName, UnitPrice
FROM Products
ORDER BY UnitPrice DESC;
```

SQL processes this query by first sorting all products by `UnitPrice` in descending order, and then returning the top three rows from that sorted list:

```
+-----------+------------------+-----------+
| ProductID | ProductName      | UnitPrice |
+-----------+------------------+-----------+
| 38        | Côte de Blaye    | 263.50    |
| 29        | Thüringer Imp.   | 123.79    |
| 9         | Mishi Kobe Niku  | 97.00     |
+-----------+------------------+-----------+
```

##### Clause Sequence with `TOP`
Notice where `TOP` sits in the query sequence: it is placed immediately after the `SELECT` keyword, before the column list:

1. `SELECT TOP (n)`
2. Column List
3. `FROM`
4. `ORDER BY`

---

#### Introductory Aggregations: MAX and COUNT

Throughout this course, you will frequently need to calculate summary statistics across entire tables. While detailed multi-group aggregations will be covered in later chapters, SQL provides built-in aggregate functions that can be used directly in a `SELECT` statement to calculate single summary metrics.

##### The `MAX()` Function
The `MAX()` function returns the highest value in a specified column. For example, to find the single highest freight charge in the `Orders` table:

```sql
SELECT MAX(Freight) AS HighestFreight
FROM Orders;
```

Notice the use of `AS HighestFreight`. The **`AS`** keyword creates an **alias**—a descriptive custom column header for the output. If you omit `AS`, SQL will display `(No column name)` for the calculated result.

##### Understanding `COUNT(*)` vs. `COUNT(column)`
The `COUNT()` function counts records, but its behavior changes depending on what is placed inside the parentheses:

* **`COUNT(*)`**: Counts **all rows** in the table, including rows that contain missing or null values in individual columns.
* **`COUNT(column_name)`**: Counts only rows where the specified column contains a **non-null (valid) value**.

Consider the `Employees` table, where some employees report to a manager (`ReportsTo`), while top executives have a `NULL` (blank) value in that field:

```sql
-- Counts all 9 employee records in the table
SELECT COUNT(*) AS TotalEmployees
FROM Employees;

-- Counts only employees who have a non-null entry in the Region column
SELECT COUNT(Region) AS EmployeesWithRegion
FROM Employees;
```

Understanding this distinction is vital for auditors when verifying whether mandatory data fields are populated across enterprise records.

---

#### Summary and Clause Sequence Checklist

In this chapter, you learned how to extract, format, sort, and sample tabular data using the SQL `SELECT` statement. 

##### The SQL Clause Execution Sequence
Programming requires strict adherence to rules. As you build queries, memorize the required order of the SQL clauses learned so far:

```sql
SELECT TOP (n) ColumnList
FROM TableName
ORDER BY ColumnToSort [ASC | DESC];
```

| Clause | Purpose | Required or Optional? |
| :--- | :--- | :--- |
| **`SELECT`** | Specifies columns or calculations to retrieve. | **Required** |
| **`TOP (n)`** | Limits the output to $n$ rows or $n\%$ of rows. | Optional (placed after `SELECT`) |
| **`FROM`** | Identifies the database source table. | **Required** |
| **`ORDER BY`** | Sorts the resulting rows in ascending or descending order. | Optional (placed after `FROM`) |

---

#### Looking Ahead
In the next section, **Chapter 3.3: Filtering Data (The `WHERE` Clause and Operators)**, we will learn how to filter data tables to extract only the rows that meet specific business criteria—such as identifying overdue invoices, out-of-stock products, or sales transactions in specific geographic territories.
