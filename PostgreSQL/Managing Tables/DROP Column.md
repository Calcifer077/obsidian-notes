---
title: PostgreSQL DROP Column
source: https://neon.com/postgresql/tutorial/drop-column
created: 2026-08-05
tags:
  - database
  - postgresql
---
To drop a column of a table, you use the `DROP COLUMN` clause in the [`ALTER TABLE`](https://neon.com/postgresql/tutorial/postgresql-alter-table) statement as follows:

```PostgreSQL
ALTER TABLE table_name
DROP COLUMN column_name;
```

When you remove a column from a table, PostgreSQL will automatically remove all of the [indexes](https://neon.com/postgresql/postgresql-indexes) and constraints that involved the dropped column.

If the column that you want to remove is used in other database objects such as [views](https://neon.com/postgresql/postgresql-views), [triggers](https://neon.com/postgresql/postgresql-triggers), and [stored procedures](https://neon.com/postgresql/postgresql-plpgsql/introduction-to-postgresql-stored-procedures), you cannot drop the column because other objects depend on it.

In this case, you can use the `CASCADE` option in the `DROP COLUMN` clause to drop the column and all of its dependent objects:

```PostgreSQL
ALTER TABLE table_name
DROP COLUMN column_name CASCADE;
```

If you remove a column that does not exists, PostgreSQL will issue an error. To remove a column if it exists only, you can use the `IF EXISTS` option as follows:

```PostgreSQL
ALTER TABLE table_name
DROP COLUMN IF EXISTS column_name;
```

In this syntax, if you remove a column that does not exist, PostgreSQL will issue a notice instead of an error.

If you want to drop multiple columns of a table simultaneously, you use multiple `DROP COLUMN` clauses in the `ALTER TABLE`  statement like this:

```PostgreSQL
ALTER TABLE table_name
DROP COLUMN column_name1,
DROP COLUMN column_name2,
...;
```

Notice that you need to add a comma (`,`) after each `DROP COLUMN` clause.

If a table has one column, you can drop it using the `ALTER TABLE...DROP COLUMN` statement. Consequently, the table will have no columns.

It’s worth noting that while PostgreSQL allows a table that has no column, it may be not allowed according to the standard SQL.

## Summary

- Use the PostgreSQL `ALTER TABLE ... DROP COLUMN` statement to drop one or more columns from a table.