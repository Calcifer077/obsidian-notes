---
title: PostgreSQL INSERT statement
source 1: https://neon.com/postgresql/tutorial/insert
source 2: https://neon.com/postgresql/tutorial/insert-multiple-rows
created: 2026-07-24
tags:
  - database
  - postgresql
---
`INSERT` allows us to insert a new row into a table. 

Basic syntax:

```PostgresQL
INSERT INTO table1(column1, column2, …)
VALUES (value1, value2, …);
```

In this syntax:
- First, specify the name of the table (`table1`) that you want to insert data after the `INSERT INTO` keywords and a list of comma-separated columns (`column1, column2, ....`).
- Second, supply a list of comma-separated values in parentheses `(value1, value2, ...)` after the `VALUES` keyword. The column and value lists must be in the same order.

The `INSERT` statement returns a command tag with the following form:

```PostgreSQL
INSERT oid count
```

In this syntax:
- The `OID` is an object identifier. PostgreSQL used the `OID` internally as a [primary key](https://neon.com/postgresql/tutorial/postgresql-primary-key) for its system tables. Typically, the `INSERT` statement returns `OID` with a value of 0.
- The `count` is the number of rows that the `INSERT` statement inserted successfully.

If we insert a row into a table successfully, the return will typically look like:

```
INSERT 0 1
```

We can also insert more than one row at a time by simply supplying more values separated by commas.

```PostgreSQL
INSERT INTO table_name (column_list)
VALUES
    (value_list_1),
    (value_list_2),
    ...
    (value_list_n);
```

Inserting multiple rows at once has advantages over inserting one row at a time:

- **Performance:** Inserting multiple rows in a single statement is often more efficient than multiple individual inserts because it reduces the number of round-trips between the application and the PostgreSQL server.
- **Atomicity:** The entire `INSERT` statement is atomic, meaning that either all rows are inserted, or none are. This ensures data consistency.

## RETURNING clause

The `INSERT` statement has an optional `RETURNING` clause that returns the information of the inserted row.

```PostgreSQL
INSERT INTO table1(column1, column2, …)
VALUES (value1, value2, …)
RETURNING id;
```

- Use `(*)` to return entire inserted row.
- We can also use alias using `AS` here to rename the returned row.

## Examples

Let's create a example table called `links`.

```PostgreSQL
CREATE TABLE links (
  id SERIAL PRIMARY KEY,
  url VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  description VARCHAR (255),
  last_update DATE
);
```

### 1) Basic INSERT example

The following example uses `INSERT` to insert a new row into the `links` table:

```postgresql
INSERT INTO links (url, name)
VALUES('https://neon.com/postgresql','PostgreSQL Tutorial');
```

- To insert character data, we need to wrap them in single quotes `('')`.
- If we omit not null columns, it will lead to a error.
- If we omit null column, it will use the column default value for insertion.
- PostgreSQL automatically generates a sequential number for the serial column so we do not have to supply a value for the serial column.

If we were to insert multiple rows at once:

```PostgreSQL
INSERT INTO contacts (first_name, last_name, email)
VALUES
    ('John', 'Doe', 'john.doe@example.com'),
    ('Jane', 'Smith', 'jane.smith@example.com'),
    ('Bob', 'Johnson', 'bob.johnson@example.com');
```

### 2) Inserting character string that contains a single quote

If we want to insert a string that contains a single quote (`'`), such as  `O'Reilly Media`, we have to use an additional single quote (`'`) to escape it.

```PostgreSQL
INSERT INTO links (url, name)
VALUES('http://www.oreilly.com','O''Reilly Media');
```


_Note: to insert into a `DATE` column, we have to use the date in format `'YYYY-MM-DD'`._

## Summary

- Use PostgreSQL `INSERT` statement to insert a new row into a table.
- Use the `RETURNING` clause to get the inserted rows.