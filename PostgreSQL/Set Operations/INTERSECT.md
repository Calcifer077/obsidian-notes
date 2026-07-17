---
title: INTERSECT operator
source: https://neon.com/postgresql/tutorial/intersect
created: 0026-07-17
tags:
  - database
  - postgresql
---
Like the [`UNION`](UNION.md) operator, `INTERSECT` operator combines result sets of two `SELECT` statements into a single result set. The `INTERSECT` operator returns a result set containing rows available in both result sets.

```PostgreSQL
SELECT select_list
FROM A
INTERSECT
SELECT select_list
FROM B;
```

To use the `INTERSECT` operator, the columns that appear in the `SELECT` statements must follow these rules:

- The number of columns and their order in queries must be the same.
- The [data types](https://neon.com/postgresql/tutorial/postgresql-data-types) of the columns in the queries must be compatible.

The following venn diagram shows the working of `INTERSECT`.

![](../../assets/PostgreSQL-INTERSECT-Operator-300x206.avif)

You can also use `ORDER BY` clause to sort the results given by `INTERSECT`.

## Example

Let's setup sample tables for examples.

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

SELECT * FROM top_rated_films;
SELECT * FROM most_popular_films;
```

A simple example to retrieve the popular films that are also top-rated (common in both tables):

```PostgreSQL
SELECT *
FROM most_popular_films
INTERSECT
SELECT *
FROM top_rated_films;
```

Output:

```
title      | release_year
-----------------+--------------
 The Godfather   |         1972
 The Dark Knight |         2008
(2 rows)
```

## Summary

- Use the PostgreSQL `INTERSECT` operator to combine two result sets and return a single result set containing rows appearing in both.
- Place the `ORDER BY` clause after the second query to sort the rows in the result set returned by the `INTERSECT` operator.