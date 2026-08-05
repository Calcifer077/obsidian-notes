---
title: PostgreSQL DROP Table
source: https://neon.com/postgresql/tutorial/drop-table
created: 2026-08-05
tags:
  - database
  - postgresql
---
To drop a table from the database, you use the `DROP TABLE` statement as follows:

```PostgreSQL
DROP TABLE [IF EXISTS] table_name
[CASCADE | RESTRICT];
```

In this syntax:

- First, specify the name of the table that you want to drop after the `DROP TABLE` keywords.
- Second, use the `IF EXISTS` option to remove the table only if it exists.

If you remove a table that does not exist, PostgreSQL issues an error. To avoid the error, you can use the `IF EXISTS` option.

If the table is used in other database objects such as [views](https://neon.com/postgresql/postgresql-views), [triggers](https://neon.com/postgresql/postgresql-triggers/enable-triggers), functions, and stored procedures, you cannot remove it. In this case, you have two options:

- Use the `CASCADE` option to remove the table and its dependent objects.
- Use the `RESTRICT` option to reject the removal if there is any object depending on the table. The `RESTRICT` option is the default if you don’t explicitly specify it in the `DROP TABLE` statement.

To remove multiple tables simultaneously, you can pass the tables separated by commas after the `DROP TABLE` keywords:

```PostgreSQL
DROP TABLE [IF EXISTS]
   table_name_1,
   table_name_2,
   ...
[CASCADE | RESTRICT];
```

_Note that you need to have the roles of the superuser, schema owner, or table owner to drop tables._

## Examples

### 1) Drop a table that has dependent objects

First, lets create two tables called `authors` and `pages`. The `pages` table has a foreign key that references the `authors` table.

```PostgreSQL
CREATE TABLE authors (
  author_id INT PRIMARY KEY,
  firstname VARCHAR (50) NOT NULL,
  lastname VARCHAR (50) NOT NULL
);

CREATE TABLE pages (
  page_id SERIAL PRIMARY KEY,
  title VARCHAR (255) NOT NULL,
  contents TEXT,
  author_id INT NOT NULL,
  FOREIGN KEY (author_id) REFERENCES authors (author_id)
);
```

The following statement uses the `DROP TABLE` to drop the `authors` table:

```PostgreSQL
DROP TABLE IF EXISTS authors;
```

Because the `authors` table has a dependent object which is a foreign key constraint in the `pages` table that references the `authors` table, PostgreSQL issues an error message:

```
ERROR:  cannot drop table authors because other objects depend on it
DETAIL:  constraint pages_author_id_fkey on table pages depends on table authors
HINT:  Use DROP ... CASCADE to drop the dependent objects too.
```

In this case, you need to remove all dependent objects first before dropping the `author` table or use `CASCADE` option as follows:

```PostgreSQL
DROP TABLE authors CASCADE;
```

This statement deletes the `authors` table as well as the constraint in the `pages` table.

If the `DROP TABLE` statement removes the dependent objects of the table that is being dropped, it will issue a notice like this:

```
NOTICE:  drop cascades to constraint pages_author_id_fkey on table pages
DROP TABLE
```

## Summary 

- Use the `DROP TABLE` statement to drop one or more tables.
- Use the `CASCADE` option to drop a table and all of its dependent objects.

