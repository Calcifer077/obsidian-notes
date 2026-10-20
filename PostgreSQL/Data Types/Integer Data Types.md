---
title: PostgreSQL Integer Data Types
source: https://neon.com/postgresql/postgresql-tutorial/postgresql-integer
created: 2026-10-17
tags:
  - postgresql
---
To store integers (positive, negative, and zero) in PostgreSQL, you can use one of the following integer types:

- `SMALLINT`
- `INTEGER`
- `BIGINT`


| **Name**   | **Storage Size** | **Min**                    | **Max**                    |
| ---------- | ---------------- | -------------------------- | -------------------------- |
| `SMALLINT` | 2 bytes          | -32,768                    | +32,767                    |
| `INTEGER`  | 4 bytes          | -2,147,483,648             | +2,147,483,647             |
| `BIGINT`   | 8 bytes          | -9,223,372,036,854,775,808 | +9,223,372,036,854,775,807 |


If you attempt to store a value outside of the permitted ranges, PostgreSQL will issue an error.

Unlike MySQL, PostgreSQL does not provide unsigned integer types (only positive and zero).

## SMALLINT 

Syntax:

```PostgreSQL
CREATE TABLE books (
    book_id SERIAL PRIMARY KEY,
    title VARCHAR (255) NOT NULL,
    pages SMALLINT NOT NULL CHECK (pages > 0)
);
```

## INTEGER 

Syntax:

```PostgreSQL
CREATE TABLE cities (
    city_id serial PRIMARY KEY,
    city_name VARCHAR (255) NOT NULL,
    population INT NOT NULL CHECK (population >= 0)
);
```

> Note that `INT` is the synonym of `INTEGER`.

## BIGINT 

Syntax:

```PostgreSQL
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    user_id INT8, -- Equivalent to BIGINT
    total_cents BIGINT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

Using `BIGINT` type is not only consuming a lot of storage but also decreasing the performance of the database, therefore, you should have a good reason to use it.

> There isn't a wide difference in performance of all three types. It just that they each have different sizes and will store different number of rows per 8KB page, so cache miss and hit will differ. For normal use case, use `INTEGER`.
## Summary 

- Use `SMALLINT`, `INT`, and `BIGINT` data types to store integers in the database. 