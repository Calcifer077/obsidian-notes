---
title: PostgreSQL Boolean Data Type
source: https://neon.com/postgresql/tutorial/boolean
created: 2026-09-29
tags:
  - postgresql
---
PostgreSQL's Boolean data type supports three values: `true`, `false` and `null`. It uses one byte for storing this value and `BOOLEAN` can be abbreviated as `BOOL`.

The following table shows the valid literal values for `TRUE` and `FALSE`:

| True   | False   |
| ------ | ------- |
| true   | false   |
| ‘t’    | ‘f ‘    |
| ‘true’ | ‘false’ |
| ‘y’    | ‘n’     |
| ‘yes’  | ‘no’    |
| ‘1’    | ‘0’     |
>Leading or trailing spaces or cases are ignored. All the constants value except for `true` and `false` must be enclosed in single quotes.

## Examples 

Let's create a sample table for examples:

```PostgreSQL
CREATE TABLE stock_availability (
   product_id INT PRIMARY KEY,
   available BOOLEAN NOT NULL
);

INSERT INTO stock_availability (product_id, available)
VALUES
  (100, TRUE),
  (200, FALSE),
  (300, 't'),
  (400, '1'),
  (500, 'y'),
  (600, 'yes'),
  (700, 'no'),
  (800, '0');
  
SELECT *
FROM stock_availability
WHERE available = 'yes';
```

Output:

```
product_id | available
------------+-----------
        100 | t
        300 | t
        400 | t
        500 | t
        600 | t
(5 rows)
```

You can imply the true value by using the Boolean column without any operator.  For example, the following query returns all available products:

```PostgreSQL
SELECT *
FROM stock_availability
WHERE available;
```

Similarly, if you want to look for `false` values, you compare the value of the Boolean column against any valid Boolean constants or use the `NOT` operator.

```PostgreSQL
SELECT
  *
FROM
  stock_availability
WHERE
  available = 'no';
  
-- OR

SELECT
  *
FROM
  stock_availability
WHERE
  NOT available;
```

> If you don't specify any value for a Boolean column `false` will be used.

## Summary 

- Use the PostgreSQL `BOOLEAN` datatype to store the boolean data.

