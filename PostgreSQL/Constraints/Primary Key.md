---
title: PostgreSQL Primary Key
source: https://neon.com/postgresql/tutorial/primary-key
created: 2026-08-10
tags:
  - database
  - postgresql
---
A primary key is a column or a group of columns used to uniquely identify a row in a table. 

A table can have zero or one primary key. It cannot have more than one primary key.

It is a good practice to add a primary key to every table. When you add a primary key to a table, PostgreSQL creates a unique B-tree index on the column or a group of columns used to define the primary key.

Technically, a primary key constraint is the combination of a [not-null constraint](https://neon.com/postgresql/tutorial/postgresql-not-null-constraint) and [a UNIQUE constraint](https://neon.com/postgresql/tutorial/postgresql-unique-constraint).

Normally, you define a primary key for a table when creating the table:

```PostgreSQL
CREATE TABLE table_name (
  column_1 data_type PRIMARY KEY,
  column_2 data_type,
  …
);
```

If the primary key consists of more than one column, you can define it using the table constraint:

```PostgreSQL
CREATE TABLE table_name (
  column_1 data_type,
  column_2 data_type,
  column_3 data_type,
  …
  PRIMARY KEY(column_1, column_2, ...)
);
```

To add a primary key to an existing table, you use the `ALTER TABLE ... ADD PRIMARY KEY` statement:

```PostgreSQL
ALTER TABLE table_name
ADD PRIMARY KEY (column_1, column_2, ...);
```

If you don't explicitly specify the name for the primary key constraint, PostgreSQL will use `table-name_pkey` as the default name.

To assign a name for the primary key, you can use the `CONSTRAINT` clause as follows:

```postgresql
CONSTRAINT constraint_name
PRIMARY KEY(column_1, column_2,...);
```

## Examples

Other examples are pretty simple and easy to understand.

### 1) Adding an auto-incremented primary key to an existing table

First, create a new table called `vendors` that does not have a primary key:

```PostgreSQL
CREATE TABLE vendors (
  name VARCHAR(255)
);
```

Second, insert some rows into the `vendors` table:

```PostgreSQL
INSERT INTO vendors (name)
VALUES
  ('Microsoft'),
  ('IBM'),
  ('Apple'),
  ('Samsung')
RETURNING *;
```

Output:

```
name
-----------
 Microsoft
 IBM
 Apple
 Samsung
(4 rows)
```

Third, add a primary key named `vendor_id` into the `vendors` table with the type `SERIAL`:

```PostgreSQL
ALTER TABLE vendors
ADD COLUMN vendor_id SERIAL PRIMARY KEY;
```

If you were to check the table:

```
vendor_id |   name
-----------+-----------
         1 | Microsoft
         2 | IBM
         3 | Apple
         4 | Samsung
(4 rows)
```

## Drop a primary key

To remove a primary key from a table, you use the following syntax:

```PostgreSQL
ALTER TABLE table_name
DROP CONSTRAINT primary_key_constraint;
```

## Summary 

- Use the `PRIMARY KEY` constraint to define a primary key for a table when creating the table.
- Use the `ALTER TABLE ... ADD PRIMARY KEY` statement to add a primary key to a table.
- Use the `ALTER TABLE ... DROP CONSTRAINT` statement to drop a primary key from a table.