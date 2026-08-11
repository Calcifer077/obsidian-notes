---
title: PostgreSQL UNIQUE Constraint
source: https://neon.com/postgresql/tutorial/unique-constraint
created: 2026-08-11
tags:
  - database
  - postgresql
---
If you want to ensure that the values stored in a column or a group of columns are unique across the whole table such as email addresses or usernames, you can use the `UNIQUE` constraint.

When a `UNIQUE` constraint is in place, every time you insert a new row or update existing data, it checks if the value is already in the table. It rejects the change and issues an error if the value already exists. 

When you add a `UNIQUE` constraint to a column or a group of columns, PostgreSQL will automatically create a _unique index_ on the column or the group of columns.

To create a `UNIQUE` constraint we can use the following syntax:

>Presented in the form of example:

```postgresql
CREATE TABLE person (
  id SERIAL PRIMARY KEY,
  email VARCHAR (50) UNIQUE
);

-- or 

CREATE TABLE person (
  id SERIAL PRIMARY KEY,
  email VARCHAR (50),
  UNIQUE(email)
);
```

We can also create unique constraint on multiple columns:

```PostgreSQL
CREATE TABLE table (
    c1 data_type,
    c2 data_type,
    c3 data_type,
    UNIQUE (c2, c3)
);
```

The combination of values in the columns `c2` and `c3` will be unique across the whole table. The value of the column `c2` or `c3` need not to be unique.

## Summary 

- Use the `UNIQUE` constraints to enforce values stored in a column or a group of columns are unique across rows within the same table.