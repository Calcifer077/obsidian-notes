---
title: PostgreSQL DELETE statement
source: https://neon.com/postgresql/tutorial/delete
created: 2026-07-24
tags:
  - database
  - postgresql
---
`DELETE` statement allows us to delete one or more rows from a table.

Basic syntax:

```PostgreSQL
DELETE FROM table_name
WHERE condition;
```

In this syntax:

- First, specify the name (`table_name`) of the table from which we want to delete data after the `DELETE FROM` keywords.
- Second, specify a condition in the [`WHERE`](../Filtering%20Data/WHERE.md) clause to determine which rows to delete.

The `WHERE` clause is optional. If we omit it, the `DELETE` statement will delete all rows in the table.

The `DELETE` statement returns the number of rows deleted. It returns zero if the `DELETE` statement did not delete any row.

We can also use `RETURNING` clause to return the deleted rows.

Note that the `DELETE` statement removes data from a table but doesn’t modify the structure of the table. If we want to change the structure of a table such as removing a column, we should use the [`ALTER TABLE`](https://neon.com/postgresql/tutorial/postgresql-alter-table) statement instead.

## Examples

Let's set up a sample table `todos`.

```PostgreSQL
CREATE TABLE todos (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    completed BOOLEAN NOT NULL DEFAULT false
);

INSERT INTO todos (title, completed) VALUES
    ('Learn basic SQL syntax', true),
    ('Practice writing SELECT queries', false),
    ('Study PostgreSQL data types', true),
    ('Create and modify tables', false),
    ('Explore advanced SQL concepts', true),
    ('Understand indexes and optimization', false),
    ('Backup and restore databases', true),
    ('Implement transactions', false),
    ('Master PostgreSQL security features', true),
    ('Build a sample application with PostgreSQL', false);
```

### 1) Deleting one row

```PostgreSQL
DELETE FROM todos
WHERE id = 1;
```

Above query deletes that row from the table which have `id` of `1`.

### 2) Deleting all the rows

```PostgreSQL
DELETE FROM todos;
```

As we didn't specify any condition, it will delete all the rows.

## Summary

- Use the `DELETE FROM` statement to delete one or more rows from a table.
- Use the `WHERE` clause to specify which rows to be deleted.
- Use the `RETURNING` clause to return the deleted rows.