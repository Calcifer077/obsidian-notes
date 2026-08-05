---
title: PostgreSQL RENAME Column
source: https://neon.com/postgresql/tutorial/rename-column
created: 2026-08-05
tags:
  - database
  - postgresql
---
To rename a column of a table, you use the `ALTER TABLE` statement with `RENAME COLUMN` clause as follows:

```PostgreSQL
ALTER TABLE table_name
RENAME COLUMN column_name TO new_column_name;
```

In this statement:

- First, specify the name of the table that contains the column which you want to rename after the `ALTER TABLE` clause.
- Second, provide the name of the column that you want to rename after the `RENAME COLUMN` keywords.
- Third, specify the new name for the column after the `TO` keyword.

The `COLUMN` keyword is optional.

If you try to rename a column that does not exist, PostgreSQL will issue an error.

To rename multiple columns, you have to execute multiple `ALTER TABLE ... RENAME COLUMN` statement.

If you rename a column referenced by other database objects such as [views](https://neon.com/postgresql/postgresql-views), [foreign key constraints](https://neon.com/postgresql/tutorial/postgresql-foreign-key), [triggers](https://neon.com/postgresql/postgresql-triggers), and [stored procedures](https://neon.com/postgresql/postgresql-plpgsql/introduction-to-postgresql-stored-procedures), PostgreSQL will automatically change the column name in the dependent objects.

## Summary 

- Use the PostgreSQL `ALTER TABLE...RENAME COLUMN` statement to rename a column.