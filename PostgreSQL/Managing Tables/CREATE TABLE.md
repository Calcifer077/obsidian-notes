---
title: PostgreSQL CREATE TABLE
source: https://neon.com/postgresql/tutorial/create-table
created: 2026-08-04
tags:
  - database
  - postgresql
---
To create a new table, you use the `CREATE TABLE` statement. Here's the basic syntax of the `CREATE TABLE` statement:

```PostgreSQL
CREATE TABLE [IF NOT EXISTS] table_name (
   column1 datatype(length) column_constraint,
   column2 datatype(length) column_constraint,
   ...
   table_constraints
);
```

In this syntax:

First, specify the name of the table that you want to create after the `CREATE TABLE` keywords. The table name must be unique in a [schema](https://neon.com/postgresql/postgresql-administration/postgresql-schema). If you create a table with a name that already exists, you’ll get an error.

A [schema](https://neon.com/postgresql/postgresql-administration/postgresql-schema) is a named collection of database objects including tables. If you create a table without a schema, it defaults to public.

Second, use the `IF NOT EXISTS` option to create a new table only if it does not exist. When you use the `IF NOT EXISTS` option and the table already exists, PostgreSQL will issue a notice instead of an error.

Third, specify table columns separated by commas. Each column definition consists of the column name, data type, size, and constraint.

The constraint of a column specifies a rule that is applied to data within a column to ensure data integrity. The column constraints include [primary key](https://neon.com/postgresql/tutorial/postgresql-primary-key), [foreign key](https://neon.com/postgresql/tutorial/postgresql-foreign-key), [not null](https://neon.com/postgresql/tutorial/postgresql-not-null-constraint), [unique](https://neon.com/postgresql/tutorial/postgresql-unique-constraint), [check](https://neon.com/postgresql/tutorial/postgresql-check-constraint), and default.

For example, the `NOT NULL` constraint ensures that the values in a column cannot be NULL.

Finally, specify constraints for the table including primary key, foreign key, and check constraints.

A table constraint is a rule that is applied to the data within the table to maintain data integrity.

Note that some column constraints can be defined as table constraints such as primary key, foreign key, unique, and check constraints.

## Constraints

PostgreSQL includes the following column constraints:

- [NOT NULL](https://neon.com/postgresql/tutorial/postgresql-not-null-constraint)– ensures that the values in a column cannot be `NULL`.
- [UNIQUE](https://neon.com/postgresql/tutorial/postgresql-unique-constraint) – ensures the values in a column are unique across the rows within the same table.
- [PRIMARY KEY](https://neon.com/postgresql/tutorial/postgresql-primary-key) – a primary key column uniquely identifies rows in a table. A table can have one and only one primary key. The primary key constraint allows you to define the primary key of a table.
- [CHECK](https://neon.com/postgresql/tutorial/postgresql-check-constraint) – ensures the data must satisfy a boolean expression. For example, the value in the price column must be zero or positive.
- [FOREIGN KEY](https://neon.com/postgresql/tutorial/postgresql-foreign-key) – ensures that the values in a column or a group of columns from a table exist in a column or group of columns in another table. Unlike the primary key, a table can have many foreign keys.

Table constraints are similar to column constraints except that you can include more than one column in the table constraint.

## Example

We will create a new table called `accounts`. The `accounts` table has the following columns:
- `user_id` – primary key
- `username` – unique and not null
- `password` – not null
- `email` – unique and not null
- `created_at` – not null
- `last_login` – null

The following example uses the `CREATE TABLE` statement to create the `accounts` table:

```PostgreSQL
CREATE TABLE accounts (
  user_id SERIAL PRIMARY KEY,
  username VARCHAR (50) UNIQUE NOT NULL,
  password VARCHAR (50) NOT NULL,
  email VARCHAR (255) UNIQUE NOT NULL,
  created_at TIMESTAMP NOT NULL,
  last_login TIMESTAMP
);
```

To create a table in a database, you need to execute the `CREATE TABLE` statement using a PostgreSQL client such as psql and pgAdmin.

In whatever tool you use you need to login user your password and execute the above command after connecting to the database under which you want to create the table.

Output:

```
Table "public.accounts"
   Column   |            Type             | Collation | Nullable |                  Default
------------+-----------------------------+-----------+----------+-------------------------------------------
 user_id    | integer                     |           | not null | nextval('accounts_user_id_seq'::regclass)
 username   | character varying(50)       |           | not null |
 password   | character varying(50)       |           | not null |
 email      | character varying(255)      |           | not null |
 created_at | timestamp without time zone |           | not null |
 last_login | timestamp without time zone |           |          |
Indexes:
    "accounts_pkey" PRIMARY KEY, btree (user_id)
    "accounts_email_key" UNIQUE CONSTRAINT, btree (email)
    "accounts_username_key" UNIQUE CONSTRAINT, btree (username)
```

## Summary 

- Use the `CREATE TABLE` statement to create a new table.
- Use the `IF NOT EXISTS` option to create the new table only if it does not exist.