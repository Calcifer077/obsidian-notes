---
title: Copying table in PostgreSQL
source: https://neon.com/postgresql/tutorial/copy-table
created: 2026-08-05
tags:
  - database
  - postgresql
---
To copy a table completely, including both table structure and data, you use the following statement:

```PostgreSQL
CREATE TABLE new_table AS
TABLE existing_table;
```

To copy a table structure without data, you add the `WITH NO DATA` clause to the `CREATE TABLE` statement as follows:

```PostgreSQL
CREATE TABLE new_table AS
TABLE existing_table
WITH NO DATA;
```

To copy a table with partial data from an existing table, you use the following statement:

```PostgreSQL
CREATE TABLE new_table AS
SELECT
*
FROM
    existing_table
WHERE
    condition;
```

_Note: All the statements above copy table structure and data but do not copy indexes and constraints of the existing table._

## Example

First, lets create a table called `contacts` and insert data into it:

```PostgreSQL
CREATE TABLE contacts(
    id SERIAL PRIMARY KEY,
    first_name VARCHAR NOT NULL,
    last_name VARCHAR NOT NULL,
    email VARCHAR NOT NULL UNIQUE
);

INSERT INTO contacts(first_name, last_name, email)
VALUES('John','Doe','john.doe@example.com'),
      ('David','William','david.william@example.com')
RETURNING *;
```

In this table, we have two _indexes_: one of primary key and another of unique.

_Note: PostgreSQL automatically creates indexes for constraints (primary key, unique, exclude only)._

Second, create a copy of `contacts` to a new table such as `contacts_backup` table:

```PostgreSQL
CREATE TABLE contact_backup
AS TABLE contacts;
```

This creates a new table with structure and data from `contacts`.

If we examine the structure of the `contact_backup` table:

```
\d contact_backup;
```

Output:

```
Table "public.contact_backup"
   Column   |       Type        | Collation | Nullable | Default
------------+-------------------+-----------+----------+---------
 id         | integer           |           |          |
 first_name | character varying |           |          |
 last_name  | character varying |           |          |
 email      | character varying |           |          |
```

Output indicates that the structure of copied table is same as original except for indexes. 

We can add the constraints using [`ALTER TABLE`](ALTER%20TABLE.md) statements:

```PostgreSQL
ALTER TABLE contact_backup ADD PRIMARY KEY(id);
ALTER TABLE contact_backup ADD UNIQUE(email);
```

If you now view the structure you will get something like this:

```
Table "public.contact_backup"
   Column   |       Type        | Collation | Nullable | Default
------------+-------------------+-----------+----------+---------
 id         | integer           |           | not null |
 first_name | character varying |           |          |
 last_name  | character varying |           |          |
 email      | character varying |           |          |
Indexes:
    "contact_backup_pkey" PRIMARY KEY, btree (id)
    "contact_backup_email_key" UNIQUE CONSTRAINT, btree (email)
```

## Summary 

- Use the `CREATE TABLE table_name AS TABLE table_copy` statement to make a copy of a table to a new one.
