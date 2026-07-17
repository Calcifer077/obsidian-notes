---
title: Union
source: https://neon.com/postgresql/tutorial/union
created: 2026-07-17
tags:
  - database
  - postgresql
---
The `UNION` operator allows us to combine the result sets of two or more [`SELECT`](../Querying%20Data/SELECT.md) statements into a single result set.

```postgresql
SELECT select_list
FROM A
UNION
SELECT select_list
FROM B;
```

In this syntax, the queries must conform to the following rules:
- The number and the order of the columns in the select list of both queries must be the same.
- The data types of the columns in select lists of the queries must be compatible.

The `UNION` operator removes all duplicates rows from the combined data set. To retain the duplicate rows, we can use `UNION ALL` instead.

Syntax of `UNION ALL` is same as `UNION` just with `ALL` after `UNION`.

The following Venn diagram illustrates how the `UNION` works:

![](../../assets/PostgresQL-UNION.avif)

The `UNION` and `UNION ALL` operators may order the rows in the result set in an unspecified order. To sort according to our use case we can use `ORDER BY` after the second `select` statement.

## Example

Let's create sample tables for examples.

```PostgreSQL
CREATE TABLE top_rated_films(
  title VARCHAR NOT NULL,
  release_year SMALLINT
);

CREATE TABLE most_popular_films(
  title VARCHAR NOT NULL,
  release_year SMALLINT
);

INSERT INTO top_rated_films(title, release_year)
VALUES
   ('The Shawshank Redemption', 1994),
   ('The Godfather', 1972),
   ('The Dark Knight', 2008),
   ('12 Angry Men', 1957);

INSERT INTO most_popular_films(title, release_year)
VALUES
  ('An American Pickle', 2020),
  ('The Godfather', 1972),
  ('The Dark Knight', 2008),
  ('Greyhound', 2020);
```

To combine data from both `top_rated_films` and `most_popular_films` we can use `UNION`.

```PostgreSQL
SELECT * FROM top_rated_films
UNION
SELECT * FROM most_popular_films;
```

Output:

```
title           | release_year
--------------------------+--------------
 An American Pickle       |         2020
 The Dark Knight          |         2008
 Greyhound                |         2020
 The Shawshank Redemption |         1994
 The Godfather            |         1972
 12 Angry Men             |         1957
(6 rows)
```

## Summary

- Use the `UNION` to combine result sets of two queries and return distinct rows.
- Use the `UNION ALL` to combine the result sets of two queries but retain the duplicate rows.