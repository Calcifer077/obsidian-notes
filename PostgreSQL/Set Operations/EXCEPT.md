---
title: EXCEPT
source: https://neon.com/postgresql/tutorial/except
created: 2026-07-17
tags:
  - database
  - postgresql
---
The `EXCEPT` operator combines rows from two result sets like [`UNION`](UNION.md) and [`INTERSECT`](INTERSECT.md) with the exception that it returns distinct rows from the first (left) query that are not in the second (right) query.

```PostgreSQL
SELECT select_list
FROM A
EXCEPT
SELECT select_list
FROM B;
```

The queries that involve the `EXCEPT` need to follow these rules:
- The number of columns and their orders must be the same in the two queries.
- The data types of the respective columns must be compatible.

The following Venn diagram illustrates the `EXCEPT` operator:

![](../../assets/PostgreSQL-EXCEPT-300x202.avif)

## Example

Let's use some sample tables for example.

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

The following statement returns the top-rated films that are not popular:

```PostgreSQL
SELECT * FROM top_rated_films
EXCEPT
SELECT * FROM most_popular_films;
```

Output:

```
title           | release_year
--------------------------+--------------
 The Shawshank Redemption |         1994
 12 Angry Men             |         1957
(2 rows)
```

## Summary

- Use the PostgreSQL `EXCEPT` operator to combine rows from two result sets and return a result set containing rows from the first result set that do not appear in the second result set.