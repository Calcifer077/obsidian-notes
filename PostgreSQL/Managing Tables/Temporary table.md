---
title: PostgreSQL Temporary Table
source: https://neon.com/postgresql/tutorial/temporary-table
created: 2026-08-05
tags:
  - database
  - postgresql
---
In PostgreSQL, a temporary table is a table that exists only during a database session. It is created and used within a single database session and is automatically dropped at the end of the session.

### Creating a temporary table

To create a temporary table, you use the `CREATE TEMPORARY TABLE` statement:

```PostgreSQL
CREATE TEMPORARY | TEMP TABLE table_name(
   column1 datatype(size) constraint,
   column2 datatype(size) constraint,
   ...,
   table_constraints
);
```

In this syntax:

- First, specify the name of the temporary table that you want to create after the `CREATE TEMPORARY TABLE` keywords. You can use either `TEMP` or `TEMPORARY` keyword.
- Second, define a list of columns for the table.

The following example creates a new temporary table `mytemp`:

```PostgreSQL
CREATE TEMP TABLE mytemp(id INT);

INSERT INTO mytemp(id) VALUES(1), (2), (3)
RETURNING *;
```

If you open a second database session and query data from `mytemp` table, you'll get an error:

You can even create a temp table with the same name as some table which is already present in the database. When querying for that table, the one with the same name as original you will get data from the temp table. You will not be able to access the permanent table until the end of the session. This also holds true for dropping a table. First, the temp table will be deleted than the original one.

To remove a temp table you can use the `DROP` statement. `DROP` statement does not have any `TEMP` keyword in it. 

## When to use temporary tables

**Isolation of data**: Since the temporary tables are session-specific, different sessions or transactions can use the same table name for temporary tables without causing a conflict. This allows you to isolate data for a specific task or session.

**Intermediate storage**: Temporary tables can be useful for storing the intermediate results of a complex query. For example, you can break down a complex query into multiple simple ones and use temporary tables as the intermediate storage for storing the partial results.

**Transaction scope**: Temporary tables can be also useful if you want to store intermediate results within a transaction. In this case, the temporary tables will be visible only to that transaction

## Summary 

- A temporary table is a short-lived table that exists during a database session or a transaction.
- Use the `CREATE TEMP TABLE` statement to create a temporary table.
- Use the `DROP TABLE` statement to drop a temporary table.