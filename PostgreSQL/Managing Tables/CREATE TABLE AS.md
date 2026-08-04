---
title: PostgreSQL CREATE TABLE AS
source: https://neon.com/postgresql/tutorial/create-table-as
created: 2026-08-04
tags:
  - database
  - postgresql
---
The `CREATE TABLE AS` statement creates a new table and fills it with the data returned by a query. The following shows the basic syntax:

```PostgreSQL
CREATE TABLE new_table_name
AS query;
```

In this syntax:
1. First, specify the new table name after the `CREATE TABLE` clause.
2. Second, provide a query whose result set is added to the new table after the `AS` keyword.

The `TEMPORARY` or `TEMP` keyword allows you to create a [temporary table](https://neon.com/postgresql/tutorial/postgresql-temporary-table):

```PostgreSQL
CREATE TEMP TABLE new_table_name
AS query;
```

The `UNLOGGED` keyword allows the new table to be created as an unlogged table:

```PostgreSQL
CREATE UNLOGGED TABLE new_table_name
AS query;
```

_Note: Unlogged table is a special type of table that bypasses the Write-Ahead Log(WAL). It improves write performance at the expense of data durability._

The columns of the new table will have the names and data types associated with the output columns of the `SELECT` clause.

If you want the table columns to have different names, you can specify the new table columns after the new table name:

```PostgreSQL
CREATE TABLE new_table_name ( column_name_list)
AS query;
```

If you want to avoid an error by creating a new table that already exists, you can use the `IF NOT EXISTS` option as follows:

```PostgreSQL
CREATE TABLE IF NOT EXISTS new_table_name
AS query;
```

## Example

Let's use `film` table from the sample database.

![](../../assets/Pasted%20image%2020260804152634.png)

The following syntax uses `CREATE TABLE` with custom column names.

```PostgreSQL
CREATE TABLE IF NOT EXISTS film_rating (rating, film_count)
AS
SELECT
    rating,
    COUNT (film_id)
FROM
    film
GROUP BY
    rating;
```

Note that the `CREATE TABLE AS` statement is similar to the [`SELECT INTO`](SELECT%20INTO.md) statement, but the `CREATE TABLE AS` statement is preferred because it is not confused with other uses of the `SELECT INTO` syntax in [PL/pgSQL](https://neon.com/postgresql/postgresql-plpgsql). In addition, the `CREATE TABLE AS` statement provides a superset of the functionality offered by the `SELECT INTO` statement.

## Summary

- Use the PostgreSQL `CREATE TABLE AS` statement to create a new table from the result of a query.