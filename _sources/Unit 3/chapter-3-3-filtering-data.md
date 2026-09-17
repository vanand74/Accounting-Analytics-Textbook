# Filtering Data: The WHERE Clause and Operators

#### Introduction: What is Filtering?
In the physical world, a filter is a device that lets certain substances pass through while blocking others—such as an air filter trapping dust or a water filter removing impurities. In data analytics, a **filter** operates on the exact same principle. A data filter evaluates rows in a table against a set of business rules, allowing matching records to pass through into your final report while blocking the rest.

Filtering is one of the most frequent and essential tasks in accounting and auditing. As an accountant or auditor, you will rarely need to view every single record in an enterprise database. Instead, you will almost always filter for specific subsets of data, such as:
* Identifying corporate credit card transactions that exceed an internal control limit (e.g., transactions greater than $10,000).
* Extracting accounts receivable balances that are more than 90 days past due.
* Finding out-of-stock inventory items that need to be reordered.
* Reviewing vendor invoices approved during non-business hours.

In SQL, filtering is accomplished using the **WHERE** clause.

---

#### The WHERE Clause Syntax
The `WHERE` clause is added to a `SELECT` statement to restrict which rows are returned by the query.

```sql
SELECT Column1, Column2, ...
FROM TableName
WHERE Condition;
```

When SQL executes a query with a `WHERE` clause, it checks the specified condition for every individual row in the target table. If the condition evaluates to **True**, that row is included in the query results. If the condition evaluates to **False**, the row is blocked.

##### Filtering Numbers
Consider the `Products` table in the *Northwind Traders* database. This table contains a column named `Discontinued`, which holds a value of `1` if a product has been discontinued and `0` if it is still actively sold by the company. 

```sql
SELECT ProductID, ProductName, UnitPrice, Discontinued
FROM Products
WHERE Discontinued = 1;
```

When you execute this query, SQL inspects every record in the `Products` table and returns only those rows where `Discontinued` equals `1`.

> **Programming Tip: Booleans in Databases**  
> In computer programming and database design, the number `1` frequently represents **True** (yes, discontinued), while `0` represents **False** (no, active).

##### Filtering Text (Strings)
When filtering columns that contain text—known in programming as **strings**—you must enclose the text value in **single quotes** (`'...'`).

For example, to list all customers located in London from the `Customers` table:

```sql
SELECT CustomerID, CompanyName, ContactName, City, Country
FROM Customers
WHERE City = 'London';
```

If you forget the single quotes around `'London'`, SQL will treat `London` as if it were a column name rather than a text value, resulting in a syntax error.

> **Note on Case Sensitivity:**  
> In Microsoft SQL Server (Transact-SQL), text comparisons in default database configurations are **not case-sensitive**. Therefore, `'London'`, `'london'`, and `'LONDON'` will return the exact same matching rows.

---

#### Comparison Operators: Asking Precise Questions
To build filtering conditions in a `WHERE` clause, you use **comparison operators**. An operator is a symbol that performs an action or evaluation on data values (just as the `+` symbol is an arithmetic operator in `5 + 9`).

Comparison operators compare two values. The result of a comparison operation is always boolean: either **True** or **False**.

| Operator | Meaning | Example | What It Tests |
| :--- | :--- | :--- | :--- |
| `=` | Equals | `WHERE Discontinued = 1` | Value equals 1 |
| `<>` | Not Equal To | `WHERE Country <> 'USA'` | Value is anything other than 'USA' |
| `>` | Greater Than | `WHERE UnitPrice > 50.00` | Price is strictly above $50.00 |
| `>=` | Greater Than or Equal To | `WHERE UnitPrice >= 15.00` | Price is $15.00 or higher |
| `<` | Less Than | `WHERE UnitsInStock < 10` | Inventory is strictly below 10 units |
| `<=` | Less Than or Equal To | `WHERE UnitsInStock <= ReorderLevel` | Inventory has reached or dropped below reorder threshold |

##### Comparison Examples
Imagine an auditor examining inventory levels and pricing in the `Products` table:

1. **Find products priced at $20.00 or higher:**
   ```sql
   SELECT ProductID, ProductName, UnitPrice
   FROM Products
   WHERE UnitPrice >= 20.00
   ORDER BY UnitPrice DESC;
   ```

2. **Find products where inventory has fallen to or below the reorder level:**
   ```sql
   SELECT ProductID, ProductName, UnitsInStock, ReorderLevel
   FROM Products
   WHERE UnitsInStock <= ReorderLevel;
   ```

---

#### Combining Conditions with Logical Operators (AND & OR)
Real-world business questions often require checking more than one condition at the same time. You can combine multiple conditions in a single `WHERE` clause using **logical operators**.

##### The Logical AND Operator
The **AND** operator requires **both** conditions to be True for a row to be included in the query results. If either condition (or both) is False, the row is blocked.

The behavior of `AND` is summarized in this truth table:

| Condition X | Condition Y | Condition X AND Condition Y |
| :---: | :---: | :---: |
| False | False | **False** |
| False | True | **False** |
| True | False | **False** |
| **True** | **True** | **TRUE** |

**Example:** Find all active (non-discontinued) products that are completely out of stock (`UnitsInStock = 0`):

```sql
SELECT ProductID, ProductName, UnitsInStock, Discontinued
FROM Products
WHERE Discontinued = 0 AND UnitsInStock = 0;
```

A product will only appear in the results if `Discontinued = 0` **and** `UnitsInStock = 0` are both True for that specific row.

##### The Logical OR Operator
The **OR** operator requires **at least one** condition to be True for a row to be included. A row is blocked only when both conditions are False.

| Condition X | Condition Y | Condition X OR Condition Y |
| :---: | :---: | :---: |
| False | False | **False** |
| False | True | **TRUE** |
| True | False | **TRUE** |
| True | True | **TRUE** |

**Example:** Find all customers located in either the United Kingdom or Germany:

```sql
SELECT CustomerID, CompanyName, City, Country
FROM Customers
WHERE Country = 'UK' OR Country = 'Germany';
```

---

#### Operator Precedence and the Power of Parentheses
When you combine `AND` and `OR` operators in the same `WHERE` clause, you must be extremely careful. SQL evaluates logical operators according to a strict priority hierarchy called **operator precedence**.

In SQL, the **AND operator has higher precedence than the OR operator**. This means SQL will always evaluate `AND` conditions before `OR` conditions, regardless of the order in which they are written in your query.

##### The Precedence Trap
Imagine your sales manager asks for a list of all suppliers who meet two specific criteria:
1. Their contact title is either `'Marketing Manager'` or `'Sales Manager'`.
2. They are located in one of four countries: `'Brazil'`, `'Canada'`, `'France'`, or `'Germany'`.

You might be tempted to write the `WHERE` clause like this:

```sql
-- CAUTION: THIS QUERY CONTAINS A LOGIC BUG DUE TO PRECEDENCE!
SELECT SupplierID, CompanyName, ContactTitle, Country
FROM Suppliers
WHERE ContactTitle = 'Marketing Manager' 
   OR ContactTitle = 'Sales Manager' 
  AND Country = 'Brazil' 
   OR Country = 'Canada' 
   OR Country = 'France' 
   OR Country = 'Germany';
```

Because `AND` takes priority, SQL groups `ContactTitle = 'Sales Manager' AND Country = 'Brazil'` together first. The resulting query will return **any** Marketing Manager anywhere in the world, plus Sales Managers in Brazil, plus **any** supplier in Canada, France, or Germany regardless of their title! This is a massive logic error.

##### Overriding Precedence with Parentheses
To fix this logic bug and override default operator precedence, **use parentheses `()`**. SQL always evaluates expressions inside parentheses first, just like in standard algebra.

```sql
-- CORRECT QUERY USING PARENTHESES
SELECT SupplierID, CompanyName, ContactTitle, Country
FROM Suppliers
WHERE (ContactTitle = 'Marketing Manager' OR ContactTitle = 'Sales Manager')
  AND (Country = 'Brazil' OR Country = 'Canada' OR Country = 'France' OR Country = 'Germany');
```

By placing parentheses around the title conditions and around the country conditions, you force SQL to evaluate the title group and country group separately before applying the `AND` operator between them.

> **The Auditor's Perspective: Logic Errors in Financial Controls**  
> In automated accounting controls, a subtle precedence error in code can leave major security gaps undetected. When writing complex queries with multiple `AND` and `OR` statements, always use parentheses to make your logical intent explicit and unambiguous.

---

#### Specialized Logical Operators: IN, BETWEEN, and LIKE
To make writing queries cleaner and more efficient, SQL provides several specialized logical operators.

##### 1. The IN Operator
Writing long chains of `OR` statements (such as checking four different countries) can quickly become tedious and hard to read. The **IN** operator provides a clean shortcut for testing whether a column's value matches any value in a specified list.

The four-country query above can be rewritten much more concisely using `IN`:

```sql
SELECT SupplierID, CompanyName, ContactTitle, Country
FROM Suppliers
WHERE (ContactTitle = 'Marketing Manager' OR ContactTitle = 'Sales Manager')
  AND Country IN ('Brazil', 'Canada', 'France', 'Germany');
```

The `IN` operator functions as an implicit set of `OR` conditions, evaluating to True if `Country` matches any element in the comma-separated list.

##### 2. The BETWEEN Operator
When you need to filter for values falling within a specific range, you can use comparison operators with `AND` (e.g., `UnitPrice >= 15.00 AND UnitPrice <= 25.00`). Alternatively, you can use the **BETWEEN** operator, which provides an inclusive range check.

```sql
SELECT ProductID, ProductName, UnitPrice
FROM Products
WHERE UnitPrice BETWEEN 15.00 AND 25.00;
```

> **Important:** The `BETWEEN` operator is **inclusive**, meaning that products priced at exactly $15.00 or $25.00 will be included in the output.

###### Using BETWEEN with Dates
The `BETWEEN` operator is especially useful when filtering dates. In SQL Server, dates are represented as strings formatted as `'YYYY-MM-DD'`:

```sql
SELECT OrderID, CustomerID, OrderDate
FROM Orders
WHERE OrderDate BETWEEN '1996-10-01' AND '1996-12-31'
ORDER BY OrderDate;
```

##### 3. The LIKE Operator (Pattern Matching)
Sometimes you need to search for text when you do not know the exact spelling or when you want to match a flexible text pattern. The **LIKE** operator allows you to perform pattern matching on text strings using **wildcard characters**.

The two primary wildcard characters in SQL are:
* `%` **(Percent sign):** Matches **zero or more** characters.
* `_` **(Underscore):** Matches exactly **one** single character.

###### Examples of Wildcard Patterns

1. **Find all customers whose contact title contains the word "Manager" anywhere in the text:**
   ```sql
   SELECT CustomerID, CompanyName, ContactTitle
   FROM Customers
   WHERE ContactTitle LIKE '%Manager%';
   ```
   *Explanation:* `%Manager%` matches `'Marketing Manager'`, `'Sales Manager'`, `'Manager'`, or `'Assistant Manager'`.

2. **Find all products whose names begin with the letter "A":**
   ```sql
   SELECT ProductID, ProductName
   FROM Products
   WHERE ProductName LIKE 'A%';
   ```

3. **Match a single specific character using underscore (`_`):**
   ```sql
   -- Matches 'cat', 'bat', 'hat', 'mat', etc.
   SELECT Word
   FROM Dictionary
   WHERE Word LIKE '_at';
   ```

4. **Match a single character in a specified range using brackets (`[]`):**
   ```sql
   -- Finds products starting with either A, B, or C
   SELECT ProductID, ProductName
   FROM Products
   WHERE ProductName LIKE '[A-C]%';
   ```

---

#### Summary and Clause Sequence Checklist

Filtering is the mechanism by which accountants transform massive database populations into targeted, actionable financial insights. 

##### The SQL Clause Execution Sequence
In SQL, clauses must be written in a strict, unchangeable order. As we add new keywords to our SQL repertoire, keep the execution sequence memorized:

```sql
SELECT TOP (n) ColumnList
FROM TableName
WHERE Condition
ORDER BY ColumnToSort [ASC | DESC];
```

| Clause | Purpose | Required or Optional? | Position in Query |
| :--- | :--- | :--- | :--- |
| **SELECT** | Specifies columns or calculations to retrieve. | **Required** | 1st Clause |
| **TOP (n)** | Limits output to `n` rows or percentage of rows. | Optional | Immediately after `SELECT` |
| **FROM** | Identifies the primary database source table. | **Required** | 2nd Clause |
| **WHERE** | Filters rows based on comparison/logical conditions. | Optional | After `FROM` |
| **ORDER BY** | Sorts the resulting output rows. | Optional | Final Clause (after `WHERE`) |

---

#### Looking Ahead
Now that you know how to extract specific columns (`SELECT`), limit sample sizes (`TOP`), sort output (`ORDER BY`), and filter rows (`WHERE`), you have mastered basic data retrieval. 

In the next section, **Chapter 3.4: Transforming Data**, we will learn how to perform calculations directly inside your queries—such as computing line-item sales revenue, calculating employee tenure, and applying string functions to clean dirty financial records on the fly.
