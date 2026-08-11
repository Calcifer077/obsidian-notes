---
title: PostgreSQL DELETE CASCADE
source: https://neon.com/postgresql/tutorial/delete-cascade
created: 2026-08-11
tags:
  - database
  - postgresql
---
The `DELETE CASCADE` is a referential action that allows you to automatically _delete_ related rows in child tables when a parent row is deleted from the parent table.

This feature helps you maintain referential integrity in the database by ensuring that dependent rows are removed when their corresponding rows are deleted. 

Syntax:

```PostgreSQL
CREATE TABLE parent_table(
    id SERIAL PRIMARY KEY,
    ...
);

CREATE TABLE child_table(
    id SERIAL PRIMARY KEY,
    parent_id INT,
    FOREIGN_KEY(parent_id)
       REFERENCES parent_table(id)
       ON DELETE CASCADE
);
```

In the child table, the `parent_id` is a foreign key that references the `id` column of the `parent_table`.

The `ON DELETE CASCADE` is the action on the [foreign key](Foreign%20Key.md) that will automatically delete the rows from the `child_table` whenever corresponding rows from the `parent_table` are deleted.

## Summary 

- Use PostgreSQL `DELETE CASCADE` action to automatically delete related rows in child tables when a parent row is deleted.

