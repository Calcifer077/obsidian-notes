---
title: PostgreSQL TRUNCATE Table
source: https://neon.com/postgresql/tutorial/truncate-table
created: 2026-08-05
tags:
  - database
  - postgresql
---
To remove all data from a table, you can use the [`DELETE`](../Modifying%20Data/DELETE.md) statement without a [`WHERE`](../Filtering%20Data/WHERE.md) clause. However, when the table has a large amount of data, the `DELETE` statement is not efficient. In this case, you can use the `TRUNCATE TABLE` statement.

The `TRUNCATE TABLE` statement deletes all data from a table very fast. Here’s the basic syntax of the `TRUNCATE TABLE` statement:

```PostgreSQL
TRUNCATE TABLE table_name;
```

- `table_name` is the name of the table that you want to truncate.

You can remove all data from multiple tables at once by separating table names by commas (`,`).

By default, the `TRUNCATE TABLE` does not remove any data from the table that has foreign key references. But you can add an additional option of `CASCADE` to delete tables that have foreign key references.

```PostgreSQL
TRUNCATE TABLE table_name
CASCADE;
```

## Restarting sequence

Besides removing data, you can also reset values of the [identity Column](Identity%20Column.md) by using the `RESTART IDENTITY` option like this:

```PostgreSQL
TRUNCATE TABLE table_name
RESTART IDENTITY;
```

By default, the  `TRUNCATE TABLE` statement uses the `CONTINUE IDENTITY` option. This option does not restart the value in the sequence associated with the column in the table.

_Note: Truncate statement does not trigger any `ON DELETE` triggers. You need to define `BEFORE TRUNCATE` and/or `AFTER TRUNCATE` triggers for that table if you wish to have triggers._

_Note: The Truncate statement is transaction safe, meaning that you can place it within a transaction._

## Why `TRUNCATE TABLE` statement is more efficient that the `DELETE` statement

The `TRUNCATE TABLE` statement is more efficient than the `DELETE` statement due to the following main reasons:

- **Minimal logging**: The `TRUNCATE TABLE` statement doesn’t generate individual row deletion logs. Instead, it deallocates entire data pages making it faster than the `DELETE` statement.
- **Fewer resources**: The truncate operation is more lightweight than the delete option because it doesn’t generate as much undo and redo information. It releases storage space without scanning individual rows.
- **Lower-level locking mechanism**: The truncate operation often requires lower-level locks and is less prone to conflicts with other transactions, which improves overall system concurrency.

## Summary 

- Use the `TRUNCATE TABLE` statement to delete all data from a large table very fast.
- Use the `CASCADE` option to truncate a table that is referenced by foreign key constraints.
- The `TRUNCATE TABLE` deletes data but does not fire `ON DELETE` triggers. Instead, it fires the `BEFORE TRUNCATE` and `AFTER TRUNCATE` triggers.
- The `TRUNCATE TABLE` statement is transaction-safe.