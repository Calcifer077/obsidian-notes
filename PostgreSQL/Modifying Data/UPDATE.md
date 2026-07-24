---
title: PostgreSQL UPDATE statement
source: https://neon.com/postgresql/tutorial/update
created: 2026-07-24
tags:
  - database
  - postgresql
---
`UPDATE` statement allows us to update data in one or more columns of one or more rows in a table.

Basic syntax:

```PostgreSQL
UPDATE table_name
SET column1 = value1,
    column2 = value2,
    ...
WHERE condition;
```

In this syntax:

- First, specify the name of the table that you want to update data after the `UPDATE` keyword.
- Second, specify columns and their new values after `SET` keyword. The columns that do not appear in the `SET` clause retain their original values.
- Third, determine which rows to update in the condition of the [`WHERE`](../Filtering%20Data/WHERE.md) clause.

The `WHERE` clause is optional. If we omit the `WHERE` clause, the `UPDATE` statement will update all rows in the table. 

When the `UPDATE` statement is executed successfully, it returns the following command tag:

```
UPDATE count
```

The `count` is the number of rows updated including _matched_ rows whose values did not change.

## Returning updated rows

The `UPDATE` statement has an optional `RETURNING` clause that returns the updated rows:

```PostgreSQL
UPDATE table_name
SET column1 = value1,
    column2 = value2,
    ...
WHERE condition
RETURNING * | output_expression AS output_name;
```

## Examples

First, let's create a sample table called `courses` and insert data into it:

```PostgreSQL
CREATE TABLE courses(
  course_id serial PRIMARY KEY,
  course_name VARCHAR(255) NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  description VARCHAR(500),
  published_date date
);


INSERT INTO courses( course_name, price, description, published_date)
VALUES
('PostgreSQL for Developers', 299.99, 'A complete PostgreSQL for Developers', '2020-07-13'),
('PostgreSQL Administration', 349.99, 'A PostgreSQL Guide for DBA', NULL),
('PostgreSQL High Performance', 549.99, NULL, NULL),
('PostgreSQL Bootcamp', 777.99, 'Learn PostgreSQL via Bootcamp', '2013-07-11'),
('Mastering PostgreSQL', 999.98, 'Mastering PostgreSQL in 21 Days', '2012-06-30');

SELECT * FROM courses;
```

### 1) Basic example

The following statement uses the `UPDATE` statement to update the course with id 3 by changing the `published_date` to `2020-08-01`.

```PostgreSQL
UPDATE courses
SET published_date = '2020-08-01'
WHERE course_id = 3;
```

It will return something like this:

```
UPDATE 1
```

`1` because only one row is updated.

### 2) Updating a column with an expression

The following statement uses an `UPDATE` statement to increase the price of all the courses by 5%.

```PostgreSQL
UPDATE courses
SET price = price * 1.05;
```

Because we don't use a `WHERE` clause, the `UPDATE` statement updates all the rows in the `courses` table.

## Summary

- Use the `UPDATE` statement to update data in one or more columns of a table.
- Specify a condition in a WHERE clause to determine which rows to update data. If no condition is specified all rows will be updated.
- Use the `RETURNING` clause to return the updated rows from the `UPDATE` statement