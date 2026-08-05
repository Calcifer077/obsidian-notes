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

## Examples

Renaming a single table is pretty simple and can be achieved by just using the above syntax.

### 1) Renaming a table that has dependent objects

If you were to rename a table on which other tables depend using foreign keys or views, PostgreSQL will automatically update the name there too.

First, create new table called `customers` and `customers_groups`:

```PostgreSQL
CREATE TABLE customer_groups(
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL
);

CREATE TABLE customers(
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    group_id INT NOT NULL,
    FOREIGN KEY (group_id) REFERENCES customer_groups(id)
       ON DELETE CASCADE
       ON UPDATE CASCADE
);
```

Here, `customers` is referencing `customer_groups` using foreign keys.

Second, create a view based on the `customers` and `customer_groups` tables:

```PostgreSQL
CREATE VIEW customer_data
AS SELECT
    c.id,
    c.name,
    g.name customer_group
FROM
    customers c
INNER JOIN customer_groups g ON g.id = c.group_id;
```

When you rename a table, PostgreSQL will automatically update its dependent objects such as [foreign key constraints](https://neon.com/postgresql/tutorial/postgresql-foreign-key), [views](https://neon.com/postgresql/postgresql-views), and [indexes](https://neon.com/postgresql/postgresql-indexes).

Third, rename the `customer_groups` table to `groups`:

```PostgreSQL
ALTER TABLE customer_groups
RENAME TO groups;
```

You can verify the foreign key constraint in the `customers` table by showing the table via `\d` command in `psql`:

```
\d customers
```

Output:

```
Table "public.customers"
  Column  |          Type          | Collation | Nullable |                Default
----------+------------------------+-----------+----------+---------------------------------------
 id       | integer                |           | not null | nextval('customers_id_seq'::regclass)
 name     | character varying(255) |           | not null |
 group_id | integer                |           | not null |
Indexes:
    "customers_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "customers_group_id_fkey" FOREIGN KEY (group_id) REFERENCES groups(id) ON UPDATE CASCADE ON DELETE CASCADE
```

Similarly, view would also be updated.

#### Why can PostgreSQL do this?

Internally, PostgreSQL doesn't store dependencies by **table name**. It stores them using an internal identifier called an **OID (Object Identifier)**.

Think of it like this:

```
Table Name: customer_groups
OID: 12345
```

After renaming:

```
Table Name: groups
OID: 12345
```

Only the **name** changes. The table's identity (OID) remains the same, so every foreign key, view, trigger, and other dependent object still points to the same table. PostgreSQL simply updates the displayed name where appropriate.

## Summary 

- Use the `ALTER TABLE ... RENAME TO` statement to rename a table.