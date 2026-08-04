---
title: PostgreSQL SELECT INTO
source: https://neon.com/postgresql/tutorial/select-into
created: 2026-08-04
tags:
  - database
  - postgresql
---
The PostgreSQL `SELECT INTO` statement creates a new table and inserts data returned from a query into the table.

The new table will have columns with the same names as the columns of the result set of the query. Unlike a regular [`SELECT`](../Querying%20Data/SELECT.md) statement, the `SELECT INTO` statement does not return a result to the client.

Here's the syntax:

```PostgreSQL
SELECT
  select_list
INTO [ TEMPORARY | TEMP ] [ TABLE ] new_table_name
FROM
  table_name
WHERE
  search_condition;
```

To create a new table with the structure and data derived from a result set, you specify the new table name after the `INTO` keyword.

The `TEMP` or `TEMPORARY` keyword is optional; it allows you to create a [temporary table](https://neon.com/postgresql/tutorial/postgresql-temporary-table) instead. Temp tables will automatically delete themselves when your current database connection closes.

The `TABLE` keyword is optional, which enhances the clarity of the statement.

Rest is just basic select statement. You could also uses joins, group by here.

Note that you cannot use the [`SELECT INTO`](https://neon.com/postgresql/postgresql-plpgsql/pl-pgsql-select-into) statement in PL/pgSQL because it interprets the `INTO` clause differently. In this case, you can use the [`CREATE TABLE AS`](https://neon.com/postgresql/tutorial/postgresql-create-table-as) statement which provides more functionality than the `SELECT INTO` statement.

## Examples

We will use `film` table for demonstration:

![](../../assets/Pasted%20image%2020260804151132.png)

### 1) Basic example

The following statement uses the `SELECT INTO` statement to create a new table called `film_r` that contains films with the rating `R` and rental duration 5 days from the `film` table.

```PostgreSQL
SELECT
    film_id,
    title,
    rental_rate
INTO TABLE film_r
FROM
    film
WHERE
    rating = 'R'
AND rental_duration = 5
ORDER BY
    title;
```

## Summary

- Use the PostgreSQL `SELECT INTO` statement to create a new table from the result set of a query.