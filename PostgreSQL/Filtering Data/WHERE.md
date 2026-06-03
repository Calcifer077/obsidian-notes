The [`SELECT`](SELECT.md) statement returns all rows from one or more columns in a table. To retrieve rows that satisfy a specified condition, you use a `WHERE` clause.

The syntax of the PostgreSQL `WHERE` clause is as follows:
```PostgreSQL
SELECT
  select_list
FROM
  table_name
WHERE
  condition
ORDER BY
  sort_expression;
```

In this syntax, you place the `WHERE` clause right after the `FROM` clause of the `SELECT` statement.

The `WHERE` clause uses the `condition` to filter the rows returned from the `SELECT` clause.

The `condition` is a boolean expression that evaluates to true, false, or unknown.

The query returns only rows that satisfy the `condition` in the `WHERE` clause. In other words, the query will include only rows that cause the `condition` to evaluate to true in the result set.

PostgreSQL evaluates the `WHERE` clause after the `FROM` clause but before the `SELECT` and `ORDER BY` clause.

If you use [Column aliases](Column%20alias.md) in the `SELECT` clause, you cannot use them in the `WHERE` clause.

Besides the `SELECT` statement, you can use the `WHERE` clause in the [`UPDATE`](https://neon.com/postgresql/tutorial/postgresql-update) and [`DELETE`](https://neon.com/postgresql/tutorial/postgresql-delete) statement to specify rows to update and delete.

To form the condition in the `WHERE` clause, you use comparison and logical operators:

| **Operator** | **Description**                                     |
| ------------ | --------------------------------------------------- |
| =            | Equal                                               |
| >            | Greater than                                        |
| <            | Less than                                           |
| >=           | Greater than or equal                               |
| <=           | Less than or equal                                  |
| <> or !=     | Not equal                                           |
| AND          | Logical operator AND                                |
| OR           | Logical operator OR                                 |
| IN           | Return true if a values matches any value in a list |
| BETWEEN      | Return true if a value is between a range of values |
| LIKE         | Return true if a value matches a pattern            |
| IS NULL      | Return true if a value is NULL                      |
| NOT          | Negate the result of the other operators            |

## Examples

We will use the `customer` table in the `dvdrental` sample database. Sample database has been loaded in [Quick Start - Settings things up](../Quick%20Start%20-%20Setting%20things%20up/Quick%20Start%20-%20Settings%20things%20up.md)
![](../../assets/customer_table_querying_data_select_statement_neon_postgre_sql_tutorial.png)

### 1) Using with equal (`=`) operator
The following statement uses the `WHERE` clause to find customers whose first name is `Jamie`:
```PostgreSQL
SELECT
  last_name,
  first_name
FROM
  customer
WHERE
  first_name = 'Jamie';
```

Output:
```
last_name | first_name
-----------+------------
 Rice      | Jamie
 Waugh     | Jamie
(2 rows)
```

### 2) Using with AND operator
The following example uses a `WHERE` clause with the `AND` logical operator to find customers whose first name and last names are `Jamie` and `Rice`:
```PostgreSQL
SELECT
  last_name,
  first_name
FROM
  customer
WHERE
  first_name = 'Jamie'
  AND last_name = 'Rice';
```

Output:
```
last_name | first_name
-----------+------------
 Rice      | Jamie
(1 row)
```

### 3) Using with the OR operator
The following example uses a `WHERE` clause with an `OR` operator to find the customers whose last name is `Rodriguez` or first name is `Adam`:
```PostgreSQL
SELECT
  first_name,
  last_name
FROM
  customer
WHERE
  last_name = 'Rodriguez'
  OR first_name = 'Adam';
```

Output:
```
first_name | last_name
------------+-----------
 Laura      | Rodriguez
 Adam       | Gooch
(2 rows)
```

### 4) Using with the IN operator
If you want to find a value in a list of values, you can use the [`IN`](https://neon.com/postgresql/tutorial/postgresql-in) operator. The following example uses the `WHERE` clause with the `IN` operator to find the customers with first names in the list Ann, Anne, and Annie:
```PostgreSQL
SELECT
  first_name,
  last_name
FROM
  customer
WHERE
  first_name IN ('Ann', 'Anne', 'Annie');
```

Output:
```
first_name | last_name
------------+-----------
 Ann        | Evans
 Anne       | Powell
 Annie      | Russell
(3 rows)
```

### 5) Using with the LIKE operator
To find a string that matches a specified pattern, you use the [`LIKE`](https://neon.com/postgresql/tutorial/postgresql-like) operator. The following example uses the `LIKE` operator in the `WHERE` clause to find customers whose first names start with the word `Ann`:
```PostgreSQL
SELECT
  first_name,
  last_name
FROM
  customer
WHERE
  first_name LIKE 'Ann%';
```

Output:
```
first_name | last_name
------------+-----------
 Anna       | Hill
 Ann        | Evans
 Anne       | Powell
 Annie      | Russell
 Annette    | Olson
(5 rows)
```

The `%` is called a wildcard that matches any string. The `'Ann%'` pattern matches any strings that start with `'Ann'`.

### 6) Using with the BETWEEN operator
The following example finds customers whose first names start with the letter `A` and contains 3 to 5 characters by using the [`BETWEEN`](https://neon.com/postgresql/tutorial/postgresql-between) operator. The `BETWEEN` operator returns true if a value is in a range of values.
```PostgreSQL
SELECT
  first_name,
  LENGTH(first_name) name_length
FROM
  customer
WHERE
  first_name LIKE 'A%'
  AND LENGTH(first_name) BETWEEN 3
  AND 5
ORDER BY
  name_length;
```

Output:
```
first_name | name_length
------------+-------------
 Amy        |           3
 Ann        |           3
 Ana        |           3
 Andy       |           4
 Anna       |           4
 Anne       |           4
 Alma       |           4
 Adam       |           4
 Alan       |           4
 Alex       |           4
 Angel      |           5
...
```

### 7) Using with the not equal operator (`<>`) 
This example finds customers whose first names start with `Bra` and last names are not `Motley`:
```PostgreSQL
SELECT
  first_name,
  last_name
FROM
  customer
WHERE
  first_name LIKE 'Bra%'
  AND last_name <> 'Motley';
```

Output:
```
first_name | last_name
------------+-----------
 Brandy     | Graves
 Brandon    | Huey
 Brad       | Mccurdy
(3 rows)
```

Note that you can use the `!=` operator and `<>` operator interchangeably because they are equivalent.

## Summary

- Use a `WHERE` clause in the `SELECT` statement to filter rows of a query based on one or more conditions.

## Sources
[Neon - where](https://neon.com/postgresql/tutorial/where)

## Tags

#database #postgresql 
