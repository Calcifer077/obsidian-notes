## Table of Contents

1. [SELECT](#1-select)
2. [WHERE](#2-where)
3. [LIKE](#3-like)
4. [ORDER BY](#4-order-by)
5. [TOP and OFFSET](#5-top-and-offset)
6. [GROUP BY](#6-group-by)
7. [HAVING](#7-having)
8. [JOINs](#8-joins)
9. [UNION](#9-union)
10. [CASE Statements](#10-case-statements)
11. [String Functions](#11-string-functions)
12. [Subqueries](#12-subqueries)
13. [CTEs (Common Table Expressions)](#13-ctes-common-table-expressions)
14. [Window Functions](#14-window-functions)
15. [Temporary Tables](#15-temporary-tables)
16. [Stored Procedures](#16-stored-procedures)

---

## 1. SELECT

The `SELECT` statement is used to query columns from a table. You can also perform calculations directly inside it.

```sql
SELECT
    first_name,
    last_name,
    age,
    age + 10         -- creates a computed column
FROM employee_demographics;
```

> **Arithmetic Order (PEMDAS):** Parentheses → Exponentiation → Multiplication → Division → Addition → Subtraction

### DISTINCT

Returns only unique values. When multiple columns are specified, it returns distinct _combinations_.

```sql
-- Unique genders only
SELECT DISTINCT gender FROM employee_demographics;

-- Unique (gender, first_name) pairs
SELECT DISTINCT gender, first_name FROM employee_demographics;
```

> **Additional tip:** You can alias computed columns using `AS`:
> 
> ```sql
> SELECT age * 2 AS double_age FROM employee_demographics;
> ```

---

## 2. WHERE

Used to filter rows before they are returned. Supports comparison operators, logical operators, and date comparisons.

```sql
-- Equality
SELECT * FROM employee_demographics WHERE first_name = 'Leslie';

-- Not equal
SELECT * FROM employee_demographics WHERE gender != 'Female';

-- Date comparison (format: YYYY-MM-DD)
SELECT * FROM employee_demographics WHERE birth_date > '1987-03-04';
```

### Logical Operators: AND, OR, NOT

```sql
-- NOT
SELECT * FROM employee_demographics WHERE NOT gender = 'Male';

-- AND
SELECT * FROM employee_demographics WHERE gender = 'Female' AND age > 30;

-- OR
SELECT * FROM employee_demographics WHERE age < 25 OR age > 55;
```

> **Additional tip:** Use parentheses to control the logic when combining AND/OR:
> 
> ```sql
> SELECT * FROM employee_demographics
> WHERE (gender = 'Female' AND age > 30) OR age < 20;
> ```

---

## 3. LIKE

Used in a `WHERE` clause to search for a specific pattern in a column.

### Wildcards

|Wildcard|Meaning|
|---|---|
|`%`|Zero, one, or multiple characters|
|`_`|Exactly one single character|

```sql
-- Starts with 'jer'
SELECT * FROM employee_demographics WHERE first_name LIKE 'jer%';

-- Contains 'er' anywhere
SELECT * FROM employee_demographics WHERE first_name LIKE '%er%';

-- Starts with 'a' followed by exactly 2 characters
SELECT * FROM employee_demographics WHERE first_name LIKE 'a__';

-- Starts with 'a', then at least 2 more characters
SELECT * FROM employee_demographics WHERE first_name LIKE 'a__%';

-- Works with dates too
SELECT * FROM employee_demographics WHERE birth_date LIKE '1989%';
```

> **Additional tip:** `LIKE` is case-insensitive in most databases (SQL Server, MySQL). Use `COLLATE` or `ILIKE` (PostgreSQL) when case sensitivity matters.

---

## 4. ORDER BY

Sorts the result set. Default order is **ascending (ASC)**; use `DESC` for descending.

```sql
-- Sort alphabetically (ascending)
SELECT * FROM employee_demographics ORDER BY first_name ASC;

-- Sort by multiple columns: first by gender, then by age within each gender
SELECT * FROM employee_demographics ORDER BY gender, age;

-- Mixed ordering per column
SELECT * FROM employee_demographics ORDER BY gender ASC, age DESC;
```

> **Tip:** You can technically use column positions (e.g., `ORDER BY 5, 4`) but this is not recommended — it breaks easily when columns are reordered.

---

## 5. TOP and OFFSET

### TOP

Retrieves a specified number of rows from the top of the result set.

```sql
SELECT TOP 3 * FROM employee_demographics;

-- Top 10 percent
SELECT TOP 10 PERCENT * FROM employee_demographics;
```

### OFFSET-FETCH (Pagination)

Must be used with `ORDER BY`. Useful for paginating large result sets.

```sql
SELECT *
FROM employee_demographics
ORDER BY employee_id
OFFSET 1 ROWS          -- skip the first row
FETCH NEXT 3 ROWS ONLY; -- return the next 3 rows
```

**How it works step by step:**

1. `ORDER BY employee_id` — sorts all rows first.
2. `OFFSET 1 ROWS` — skips the 1st row.
3. `FETCH NEXT 3 ROWS ONLY` — returns rows 2, 3, and 4.

> **Additional tip:** For page-based pagination, use:
> 
> ```sql
> OFFSET (@page_number - 1) * @page_size ROWS
> FETCH NEXT @page_size ROWS ONLY;
> ```

---

## 6. GROUP BY

Groups rows that share the same value in specified columns into summary rows. Commonly paired with aggregate functions.

### Aggregate Functions

|Function|Description|
|---|---|
|`COUNT()`|Number of rows|
|`SUM()`|Total of values|
|`AVG()`|Average of values|
|`MAX()`|Maximum value|
|`MIN()`|Minimum value|

```sql
-- Average age per gender
SELECT gender, AVG(age)
FROM employee_demographics
GROUP BY gender;

-- Multiple aggregates
SELECT gender, COUNT(*) AS total, AVG(age) AS avg_age, MAX(age) AS oldest
FROM employee_demographics
GROUP BY gender;
```

> **Rule:** Any column in `SELECT` that is **not** inside an aggregate function **must** appear in `GROUP BY`.

---

## 7. HAVING

Filters _after_ grouping — used when you want to apply conditions on aggregated values. `WHERE` cannot be used with aggregate functions because it filters before grouping happens.

```sql
-- Wrong: WHERE can't filter on aggregated result
SELECT gender, AVG(age)
FROM employee_demographics
WHERE AVG(age) > 40   -- ERROR
GROUP BY gender;

-- Correct: use HAVING
SELECT gender, AVG(age)
FROM employee_demographics
GROUP BY gender
HAVING AVG(age) > 40;
```

### Using Both WHERE and HAVING

```sql
-- First filter rows (WHERE), then filter groups (HAVING)
SELECT occupation, AVG(salary)
FROM employee_salary
WHERE occupation LIKE '%manager'    -- row-level filter
GROUP BY occupation
HAVING AVG(salary) > 60000;        -- group-level filter
```

> **Summary:** `WHERE` filters rows → `GROUP BY` groups them → `HAVING` filters groups.

---

## 8. JOINs

Used to combine columns from two or more tables based on a related column.

### INNER JOIN

Returns only rows that have matching values in **both** tables.

```sql
SELECT demo.first_name, age, occupation
FROM employee_demographics AS demo
INNER JOIN employee_salary AS sal
    ON demo.employee_id = sal.employee_id;
```

> **Ambiguity:** If the same column name exists in both tables (e.g., `employee_id`), always prefix with the table alias to avoid errors.

### LEFT JOIN

Returns **all rows from the left table** and matching rows from the right. Unmatched rows from the right return `NULL`.

```sql
SELECT *
FROM employee_demographics AS demo
LEFT JOIN employee_salary AS sal
    ON demo.employee_id = sal.employee_id;
```

### RIGHT JOIN

Returns **all rows from the right table** and matching rows from the left. Unmatched rows from the left return `NULL`.

```sql
SELECT *
FROM employee_demographics AS demo
RIGHT JOIN employee_salary AS sal
    ON demo.employee_id = sal.employee_id;
```

### FULL OUTER JOIN _(additional)_

Returns all rows from **both** tables. Unmatched rows from either side return `NULL`.

```sql
SELECT *
FROM employee_demographics AS demo
FULL OUTER JOIN employee_salary AS sal
    ON demo.employee_id = sal.employee_id;
```

### SELF JOIN

A table joined with itself. Useful for comparing rows within the same table (e.g., finding pairs, hierarchy).

```sql
SELECT *
FROM employee_salary AS emp1
JOIN employee_salary AS emp2
    ON emp1.employee_id + 1 = emp2.employee_id;
```

### Joining Multiple Tables

```sql
SELECT *
FROM employee_demographics AS dem
INNER JOIN employee_salary AS sal
    ON dem.employee_id = sal.employee_id
INNER JOIN parks_departments AS pd
    ON sal.dept_id = pd.department_id;
```

### JOIN Comparison Summary

|Join Type|Returns|
|---|---|
|`INNER JOIN`|Only matching rows in both tables|
|`LEFT JOIN`|All left rows + matched right rows|
|`RIGHT JOIN`|All right rows + matched left rows|
|`FULL OUTER JOIN`|All rows from both tables|
|`SELF JOIN`|Rows from the same table compared with each other|

---

## 9. UNION

Combines **rows** from two or more queries into a single result set.

**Rules:**

- The number of columns must be the same in all queries.
- The data types of corresponding columns must be compatible.
- Column names in the result come from the first query.

```sql
-- UNION: returns unique rows only (duplicates removed)
SELECT first_name, last_name FROM employee_demographics
UNION
SELECT first_name, last_name FROM employee_salary;

-- UNION ALL: includes all rows including duplicates
SELECT first_name, last_name FROM employee_demographics
UNION ALL
SELECT first_name, last_name FROM employee_salary;
```

### Practical UNION Example — Labeling Groups

```sql
SELECT first_name, last_name, 'Old Man' AS Label
FROM employee_demographics
WHERE age > 40 AND gender = 'Male'

UNION

SELECT first_name, last_name, 'Old Lady' AS Label
FROM employee_demographics
WHERE age > 40 AND gender = 'Female'

UNION

SELECT first_name, last_name, 'Highly Paid Employee' AS Label
FROM employee_salary
WHERE salary > 70000;
```

> **UNION vs JOIN:** `JOIN` combines _columns_ horizontally; `UNION` stacks _rows_ vertically.

---

## 10. CASE Statements

Adds conditional logic (`IF/ELSE`) directly in SQL. The first matching `WHEN` condition is executed.

```sql
-- Basic CASE
SELECT first_name, last_name, age,
    CASE
        WHEN age <= 30 THEN 'Young'
        WHEN age BETWEEN 31 AND 50 THEN 'Middle-aged'
        WHEN age > 50 THEN 'Senior'
    END AS age_bracket
FROM employee_demographics;
```

### Multiple CASE Statements in One Query

```sql
SELECT first_name, last_name, salary,
    CASE
        WHEN salary < 50000 THEN salary * 1.05
        WHEN salary > 50000 THEN salary * 1.07
        ELSE salary            -- no change if exactly 50000
    END AS new_salary,
    CASE
        WHEN dept_id = 6 THEN salary * 0.10
        ELSE 0
    END AS bonus
FROM employee_salary;
```

> **NULL behavior:** If a row doesn't match any `WHEN` condition and there's no `ELSE`, the result is `NULL`. Always add an `ELSE` clause when you want to avoid nulls — but ensure the data type is compatible with other branches.

---

## 11. String Functions

A collection of built-in functions to manipulate text data.

### Length

```sql
SELECT LEN('mahesh');   -- Returns 6

SELECT first_name, LEN(first_name) AS name_length
FROM employee_demographics;
```

### Case Conversion

```sql
SELECT UPPER('sky');    -- 'SKY'
SELECT LOWER('sKy');    -- 'sky'
```

### Trimming Whitespace

```sql
SELECT TRIM('    dk ');     -- 'dk'   (both ends)
SELECT RTRIM('    dk   ');  -- '    dk' (right only)
SELECT LTRIM('    dk    '); -- 'dk    ' (left only)
```

### LEFT / RIGHT

Extract characters from the beginning or end of a string.

```sql
SELECT first_name, LEFT(first_name, 4), RIGHT(first_name, 4)
FROM employee_demographics;
```

### SUBSTRING

Extract a portion of a string: `SUBSTRING(string, start_position, length)`. Index is **1-based**.

```sql
SELECT first_name, SUBSTRING(first_name, 3, 2)
FROM employee_demographics;
-- Starting at position 3, takes 2 characters
```

### REPLACE

Replaces all occurrences of a substring. **Case-sensitive.**

```sql
SELECT first_name, REPLACE(first_name, 'a', 'z')
FROM employee_demographics;
```

### CHARINDEX

Searches for an expression and returns its starting position. Optional third argument sets the start of the search.

```sql
SELECT CHARINDEX('x', 'Alexxxander');       -- Returns 4
SELECT CHARINDEX('x', 'Alexxxander', 5);    -- Search starts from position 5
```

### CONCAT

Joins multiple strings together.

```sql
SELECT first_name, last_name,
    CONCAT(first_name, ' ', last_name) AS full_name
FROM employee_demographics;
```

> **Additional tip:** In SQL Server you can also use `+` for string concatenation: `first_name + ' ' + last_name`. However, if any value is `NULL`, the whole result becomes `NULL` — `CONCAT` handles nulls more gracefully by treating them as empty strings.

---

## 12. Subqueries

A query nested inside another query, enclosed in parentheses.

### Subquery in WHERE

```sql
-- Get employees whose IDs appear in dept 1 of salary table
SELECT *
FROM employee_demographics
WHERE employee_id IN (
    SELECT employee_id
    FROM employee_salary
    WHERE dept_id = 1
);
```

### Subquery in SELECT

```sql
-- Show each employee's salary alongside the overall average
SELECT first_name, salary,
    (SELECT AVG(salary) FROM employee_salary) AS avg_salary
FROM employee_salary;
```

### Subquery in FROM

Used when you need to aggregate over an already-aggregated result.

```sql
SELECT MAX(max_age)
FROM (
    SELECT
        gender,
        AVG(age) AS avg_age,
        MAX(age) AS max_age,
        MIN(age) AS min_age,
        COUNT(age) AS count_age
    FROM employee_demographics
    GROUP BY gender
) AS agg_table;
```

> **Subquery vs CTE:** Subqueries are inline and can be harder to read for complex queries. CTEs (see next section) offer the same power with better readability and reusability.

---

## 13. CTEs (Common Table Expressions)

A CTE is a named, temporary result set defined using the `WITH` keyword. It exists only for the duration of the query it's attached to.

**Key rules:**

- CTEs must be used _immediately_ after definition — you can't define a CTE and run it later.
- You can reference aliases from the inner query in the outer query.
- You can apply further aggregation on top of a CTE's result (e.g., `AVG(avg_sal)`).

### Basic CTE

```sql
WITH cte_example AS (
    SELECT
        gender,
        AVG(salary) AS avg_sal,
        MAX(salary) AS max_sal,
        MIN(salary) AS min_sal,
        COUNT(salary) AS count_sal
    FROM employee_demographics AS dem
    JOIN employee_salary AS sal
        ON dem.employee_id = sal.employee_id
    GROUP BY gender
)
SELECT *
FROM cte_example;
```

### Multiple CTEs

Chain multiple CTEs by separating them with a comma.

```sql
WITH cte_demographics AS (
    SELECT employee_id, gender, birth_date
    FROM employee_demographics
    WHERE birth_date > '1985-01-01'
),
cte_salary AS (
    SELECT employee_id, salary
    FROM employee_salary
    WHERE salary > 50000
)
SELECT *
FROM cte_demographics
JOIN cte_salary
    ON cte_demographics.employee_id = cte_salary.employee_id;
```

### CTE with Custom Column Aliases

Aliases defined in the parentheses next to the CTE name override those inside the query body.

```sql
WITH cte_example (Gender, Avg_sal, Max_sal, Min_sal, Count_sal) AS (
    SELECT
        gender,
        AVG(salary),
        MAX(salary),
        MIN(salary),
        COUNT(salary)
    FROM employee_demographics AS dem
    JOIN employee_salary AS sal
        ON dem.employee_id = sal.employee_id
    GROUP BY gender
)
SELECT Avg_sal
FROM cte_example;
```

> **CTE vs Subquery vs Temp Table:**
> 
> - **CTE** — best for readability and when you need to reference the result once.
> - **Subquery** — quick and inline, but can get deeply nested.
> - **Temp Table** — best when you need to query the intermediate result multiple times or manipulate it further.

---

## 14. Window Functions

Window functions perform calculations across a _set of rows related to the current row_ **without collapsing the result** — unlike `GROUP BY`. Each row keeps its individual detail while also gaining an aggregate or ranking value.

### Aggregate Window Functions

```sql
-- AVG salary per gender, shown alongside each employee's details
SELECT
    dem.first_name,
    dem.last_name,
    gender,
    AVG(salary) OVER (PARTITION BY gender) AS avg_salary_by_gender
FROM employee_demographics dem
JOIN employee_salary sal
    ON dem.employee_id = sal.employee_id;
```

`PARTITION BY` divides rows into groups (like `GROUP BY`), but each row is still returned individually.

### Rolling Total (Running Sum)

```sql
SELECT
    dem.first_name,
    dem.last_name,
    gender,
    salary,
    SUM(salary) OVER (PARTITION BY gender ORDER BY dem.employee_id) AS rolling_total
FROM employee_demographics dem
JOIN employee_salary sal
    ON dem.employee_id = sal.employee_id;
```

> Adding `ORDER BY` inside `OVER()` turns the aggregate into a _cumulative_ (running) calculation — like a prefix sum per group.

### Ranking Functions

|Function|Behavior|
|---|---|
|`ROW_NUMBER()`|Unique sequential number for every row (no ties)|
|`RANK()`|Same rank for ties; skips subsequent rank numbers|
|`DENSE_RANK()`|Same rank for ties; does **not** skip rank numbers|

**Example comparison:**

|salary|ROW_NUMBER|RANK|DENSE_RANK|
|---|---|---|---|
|100|1|1|1|
|90|2|2|2|
|90|3|2|2|
|80|4|4|3|

```sql
SELECT
    dem.first_name,
    dem.last_name,
    gender,
    salary,
    ROW_NUMBER()  OVER (PARTITION BY gender ORDER BY salary DESC) AS row_num,
    RANK()        OVER (PARTITION BY gender ORDER BY salary DESC) AS row_rank,
    DENSE_RANK()  OVER (PARTITION BY gender ORDER BY salary DESC) AS dense_rank
FROM employee_demographics dem
JOIN employee_salary sal
    ON dem.employee_id = sal.employee_id;
```

### ROW_NUMBER — Without Partition

```sql
SELECT
    dem.first_name,
    dem.last_name,
    salary,
    ROW_NUMBER() OVER (ORDER BY dem.employee_id) AS row_num
FROM employee_demographics dem
JOIN employee_salary sal
    ON dem.employee_id = sal.employee_id;
```

> **Additional tip — LAG / LEAD:** Two other useful window functions for accessing adjacent rows:
> 
> ```sql
> -- Previous row's salary
> LAG(salary, 1) OVER (ORDER BY employee_id) AS prev_salary
> 
> -- Next row's salary
> LEAD(salary, 1) OVER (ORDER BY employee_id) AS next_salary
> ```

---

## 15. Temporary Tables

Temporary tables are session-scoped storage areas for holding intermediate results. They are visible in the schema but exist only in memory for the session.

### Local Temp Table (`#`)

Visible only to the current session. Automatically dropped when the session ends.

```sql
CREATE TABLE #temp_table (
    first_name      VARCHAR(50),
    last_name       VARCHAR(50),
    favorite_movie  VARCHAR(100)
);

-- Insert data
INSERT INTO #temp_table VALUES ('Alex', 'Freberg', 'LOTR');

-- Query it
SELECT * FROM #temp_table;
```

### Global Temp Table (`##`)

Visible to **all sessions**. Dropped when the creating session ends and no other session is referencing it.

```sql
CREATE TABLE ##global_temp_table (
    first_name      VARCHAR(50),
    last_name       VARCHAR(50),
    favorite_movie  VARCHAR(100)
);
```

### Creating a Temp Table from a Query (`SELECT INTO`)

```sql
SELECT *
INTO #salary_over_50k
FROM employee_salary
WHERE salary >= 50000;

SELECT * FROM #salary_over_50k;
```

> **Tip:** If you need more control over the table structure, define it first with `CREATE TABLE`, then use `INSERT INTO ... SELECT` to populate it. The `SELECT INTO` shortcut is convenient but doesn't let you pre-define data types, constraints, or indexes.

---

## 16. Stored Procedures

Stored procedures are reusable blocks of SQL code — like functions — that are saved in the database and can be executed later with `EXEC`.

### Simple Stored Procedure

```sql
CREATE PROCEDURE large_salaries
AS
BEGIN
    SELECT * FROM employee_salary
    WHERE salary >= 50000;
END;
GO

EXEC large_salaries;
```

### Multiple Queries in One Procedure

```sql
CREATE PROCEDURE large_salaries2
AS
BEGIN
    SELECT * FROM employee_salary WHERE salary >= 50000;
    SELECT * FROM employee_salary WHERE salary >= 10000;
END;

EXEC large_salaries2;
```

### Stored Procedure with Parameters

```sql
CREATE OR ALTER PROCEDURE get_salary_from_employee_id
    @employee_id INT
AS
BEGIN
    SELECT salary
    FROM employee_salary
    WHERE employee_id = @employee_id;
END;

EXEC get_salary_from_employee_id @employee_id = 1;
```

> **Additional tips:**
> 
> - Use `CREATE OR ALTER` (SQL Server 2016+) to avoid dropping and recreating the procedure when making changes.
> - You can have default parameter values:
>     
>     ```sql
>     @min_salary INT = 50000
>     ```
>     
> - Use `TRY...CATCH` inside procedures for proper error handling:
>     
>     ```sql
>     BEGIN TRY    -- your logicEND TRYBEGIN CATCH    PRINT ERROR_MESSAGE();END CATCH
>     ```
>     

---

## Quick Reference — SQL Execution Order

SQL statements are _written_ in one order but _executed_ in a different logical order:

|Step|Clause|What it does|
|---|---|---|
|1|`FROM` / `JOIN`|Identify source tables|
|2|`WHERE`|Filter rows|
|3|`GROUP BY`|Group remaining rows|
|4|`HAVING`|Filter groups|
|5|`SELECT`|Pick columns / compute expressions|
|6|`DISTINCT`|Remove duplicates|
|7|`ORDER BY`|Sort result|
|8|`TOP` / `OFFSET-FETCH`|Limit rows returned|

Understanding this order explains why `WHERE` can't use aggregate functions (grouping hasn't happened yet) and why you can't reference a `SELECT` alias in `WHERE` (SELECT runs after WHERE).
