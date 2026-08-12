---
title: PostgreSQL NOT NULL Constraint
source: https://neon.com/postgresql/tutorial/not-null-constraint
created: 2026-08-12
tags:
  - database
  - postgresql
---
In database, `NULL` represents unknown or missing information. `NULL` is not the same as an empty string or the number zero. 

Suppose you need to insert the email address of a contact into a table. You may request the user's email address. However you may not know user's email address so you can insert `NULL` indicating that the email address is unknown at recording time. 

`NULL` does not equate to anything not even itself. 

The expression `NULL = NULL` returns `NULL`.

To check if a value is `NULL` or not, you use the `IS NULL` boolean operator. 

```PostgreSQL
email_address IS NULL
```

The `IS NOT NULL` operator negates the result of the `IS NULL` operator.

To control whether a column can accept `NULL`, you use the `NOT NULL` constraint:

```PostgreSQL
CREATE TABLE table_name(
   ...
   column_name data_type NOT NULL,
   ...
);
```

If a column has a `NOT NULL` constraint, any attempt to insert or update `NULL` in the column will result in an error.

## Adding `NOT NULL` constraints to existing columns 

To add the `NOT NULL` constraint to a column of an existing table we can use `ALTER TABLE`  statement.

```PostgreSQL
ALTER TABLE table_name
ALTER COLUMN column_name SET NOT NULL;
```

To add `NOT NULL` constraint to multiple columns, you use the following syntax:

```PostgreSQL
ALTER TABLE table_name
ALTER COLUMN column_name_1 SET NOT NULL,
ALTER COLUMN column_name_2 SET NOT NULL,
...;
```

## Special case of `NOT NULL` constraint 

Besides the `NOT NULL` constraint, you can use [CHECK constraint](CHECK.md) to force a column to accept not `NULL` values.

```PostgreSQL
CHECK(column IS NOT NULL)
```

This is useful because sometimes you may want either column `a` or `b` is not null, but not both.

For example, you may want either `username` or `email` column of the user tables is not null or empty. In this case, you can use the `CHECK` constraint as follows:

```PostgreSQL
CREATE TABLE users (
  id serial PRIMARY KEY,
  username VARCHAR (50),
  password VARCHAR (50),
  email VARCHAR (50),
  CONSTRAINT username_email_notnull CHECK (
    NOT (
      (
        username IS NULL
        OR username = ''
      )
      AND (
        email IS NULL
        OR email = ''
      )
    )
  )
);
```

The following statement works.

```PostgreSQL
INSERT INTO users (username, email)
VALUES
	('user1', NULL),
	(NULL, 'user2@example.com'),
	('user2', 'user2@example.com'),
	('user3', '');
```

However, the following statement will not work.

```PostgreSQL
INSERT INTO users (username, email)
VALUES
	(NULL, NULL),
	(NULL, ''),
	('', NULL),
	('', '');
```

## Summary 

- Use the `NOT NULL` constraint for a column to enforce a column not accept `NULL`. By default, a column can hold NULL.
- To check if a value is `NULL` or not, you use the `IS NULL` operator. The `IS NOT NULL` negates the result of the `IS NULL`.
- Never use equal operator `=` to compare a value with `NULL` because it always returns `NULL`.