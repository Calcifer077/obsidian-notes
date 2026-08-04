---
title: PostgreSQL ALTER TABLE
source: https://neon.com/postgresql/tutorial/alter-table
created: 2026-08-04
tags:
  - database
  - postgresql
---
To change the structure of an existing table, you use PostgreSQL `ALTER TABLE` statement.

The following illustrates the basic syntax of the `ALTER TABLE` statement:

```PostgreSQL
ALTER TABLE table_name action;
```

PostgreSQL provides you with many actions:

- [Add a column](https://neon.com/postgresql/tutorial/postgresql-add-column)
- [Drop a column](https://neon.com/postgresql/tutorial/postgresql-drop-column)
- [Change the data type of a column](https://neon.com/postgresql/tutorial/postgresql-change-column-type)
- [Rename a column](https://neon.com/postgresql/tutorial/postgresql-rename-column)
- [Set a default value for the column](https://neon.com/postgresql/tutorial/postgresql-default-value)
- Add a constraint to a column.
- [Rename a table](https://neon.com/postgresql/tutorial/postgresql-rename-table)

To add a new column to a table, you use [`ALTER TABLE ADD COLUMN`](https://neon.com/postgresql/tutorial/postgresql-add-column) statement:

```PostgreSQL
ALTER TABLE table_name
ADD COLUMN column_name datatype column_constraint;
```

To drop a column from a table, you can use [`ALTER TABLE DROP COLUMN`](https://neon.com/postgresql/tutorial/postgresql-drop-column) statement:

```PostgreSQL
ALTER TABLE table_name
DROP COLUMN column_name;
```

To rename a column, you can use `ALTER TABLE RENAME COLUMN` statement:

```PostgreSQL
ALTER TABLE table_name
RENAME COLUMN column_name
TO new_column_name;
```

To change a default value of the column, you use `ALTER TABLE ALTER COLUMN SET` or `DROP DEFAULT`:

```PostgreSQL
ALTER TABLE table_name
ALTER COLUMN column_name
[SET DEFAULT value | DROP DEFAULT];
```

To change the [`NOT NULL` constraint](https://neon.com/postgresql/tutorial/postgresql-not-null-constraint), you use `ALTER TABLE ALTER COLUMN` statement:

```PostgreSQL
ALTER TABLE table_name
ALTER COLUMN column_name
[SET NOT NULL| DROP NOT NULL];
```

To add a `CHECK` constraint, you use `ALTER TABLE ADD CHECK` statement:

```PostgreSQL
ALTER TABLE table_name
ADD CHECK expression;
```

Generally, to add a constraint to a table, you use `ALTER TABLE ADD CONSTRAINT` statement:

```PostgreSQL
ALTER TABLE table_name
ADD CONSTRAINT constraint_name constraint_definition;
```

To [rename a table](https://neon.com/postgresql/tutorial/postgresql-rename-table) you use `ALTER TABLE RENAME TO` statement:

```PostgreSQL
ALTER TABLE table_name
RENAME TO new_table_name;
```