The `SELECT DISTINCT` removes duplicate rows from a result set. This clause retains one row for each group of duplicates. It can be applied to one or more columns in the select list of the `SELECT` statement.

Syntax:
```PostgreSQL
SELECT
  DISTINCT column1
FROM
  table_name;
```

Above the `SELECT DISTINCT` uses the values in the `column1` column to evaluate the duplicate.

If you specify multiple columns, the `SELECT DISTINCT` clause will evaluate the duplicate based on the combination of values in these columns. For example:
```PostgreSQL
SELECT
   DISTINCT column1, column2
FROM
   table_name;
```

Note that PostgreSQL also offers the [DISTINCT ON](DISTINCT%20ON.md) clause that retains the first unique entry of a column or combination of columns in the result set.

If you want to find distinct values of all columns in a table, you can use `SELECT DISTINCT *`:
```PostgreSQL
SELECT DISTINCT *
FROM table_name;
```

## Examples

First let's create a table for examples:
```PostgreSQL
-- CREATE TABLE
CREATE TABLE colors(
  id SERIAL PRIMARY KEY,
  bcolor VARCHAR,
  fcolor VARCHAR
);

-- INSERT SOME ROWS
INSERT INTO
  colors (bcolor, fcolor)
VALUES
  ('red', 'red'),
  ('red', 'red'),
  ('red', NULL),
  (NULL, 'red'),
  (NULL, NULL),
  ('green', 'green'),
  ('blue', 'blue'),
  ('blue', 'blue');
```

What the table looks like:
```PostgreSQL
SELECT
  id,
  bcolor,
  fcolor
FROM
  colors;
```

Output:
```
id | bcolor | fcolor
----+--------+--------
  1 | red    | red
  2 | red    | red
  3 | red    | null
  4 | null   | red
  5 | null   | null
  6 | green  | green
  7 | blue   | blue
  8 | blue   | blue
(8 rows)
```

### 1) On one column
The following statement selects unique values from the `bcolor` column of the `colors` table and [sorts](ORDER%20BY.md) the result set in alphabetical order by using the [`ORDER BY`](ORDER%20BY.md) clause.
```PostgreSQL
SELECT
  DISTINCT bcolor
FROM
  colors
ORDER BY
  bcolor;
```

Output:
```
bcolor
--------
 blue
 green
 red
 null
(4 rows)
```

The `bcolor` column has three 'red' entries, two `NULL`, one 'green', and two 'blue'. `SELECT DISTINCT` removes two 'red' values, one `NULL`, and one 'blue'.

Note that PostgreSQL treats `NULL`s as duplicates so that it keeps one `NULL` for all `NULL`s when you apply the `SELECT DISTINCT` clause.

### 2) On multiple columns
```PostgreSQL
SELECT
  DISTINCT bcolor, fcolor
FROM
  colors
ORDER BY
  bcolor,
  fcolor;
```

Output:
```
bcolor | fcolor
--------+--------
 blue   | blue
 green  | green
 red    | red
 red    | null
 null   | red
 null   | null
(6 rows)
```

In this example, the query uses the values from both `bcolor` and `fcolor` (as a combination) columns to evaluate the uniqueness of rows.

## Summary

- Use the `SELECT DISTINCT` to remove duplicate rows from a result set of a query.

## Sources

[Neon - select distinct](https://neon.com/postgresql/tutorial/select-distinct)

## Tags

#database 
#postgresql 

