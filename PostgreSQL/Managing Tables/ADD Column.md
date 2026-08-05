---
title: PostgreSQL ADD Column
source: https://neon.com/postgresql/tutorial/add-column
created: 2026-08-05
tags:
  - database
  - postgresql
---
To add a new column to an existing table, you use the [`ALTER TABLE`](https://neon.com/postgresql/tutorial/postgresql-alter-table) `ADD COLUMN` statement as follows:

```PostgreSQL
ALTER TABLE table_name
ADD COLUMN new_column_name data_type constraint;
```

In this syntax:

- First, specify the name of the table to which you want to add a new column after the `ALTER TABLE` keyword.
- Second, specify the name of the new column as well as its data type and constraint after the `ADD COLUMN` keywords.

When you add a new column to the table, PostgreSQL appends it at the end of the table. PostgreSQL has no option to specify the position of the new column in the table.

To add multiple columns to an existing table, you use multiple `ADD COLUMN` clauses in the `ALTER TABLE` statement as follows:

```PostgreSQL
ALTER TABLE table_name
ADD COLUMN column_name1 data_type constraint,
ADD COLUMN column_name2 data_type constraint,
...
ADD COLUMN column_namen data_type constraint;
```

The examples are pretty simple and easy to understand. So skipping those.
## Summary

- Use the PostgreSQL `ALTER TABLE...ADD COLUMN` statement to add one or more columns to a table.
  