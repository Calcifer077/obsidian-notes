---
title: PostgreSQL Rename Table
source: https://neon.com/postgresql/tutorial/rename-table
created: 2026-08-04
tags:
  - database
  - postgresql
---
To change the name of an existing table, you can use the `ALTER TABLE ... RENAME TO` statement as follows:

```PostgreSQL
ALTER TABLE table_name
RENAME TO new_table_name;
```

In this statement:

- First, specify the name of the table which you want to rename after the `ALTER TABLE` clause.
- Second, assign the new table name after the `RENAME TO` clause.

If you rename a table that does not exists, PostgreSQL will issue an error.

To avoid the error, you can use the `IF EXISTS` option:

```PostgreSQL
ALTER TABLE IF EXISTS table_name
RENAME TO new_table_name;
```

In this case, if the `table_name` does not exist, PostgreSQL will issue a notice instead.

