---
title: PostgreSQL Array Data Type
source: https://neon.com/postgresql/tutorial/array
created: 2026-10-24
tags:
  - postgresql
---
An array is a collection of elements that have the same data type.

Arrays can be one-dimensional, multidimensional, or even nested arrays.

Every data type has its companion array type e.g., `integer` has `integer[]` array type, and same for rest.

If you define a [user-defined data type](https://neon.com/postgresql/tutorial/postgresql-user-defined-data-types), PostgreSQL also creates a corresponding array type automatically for you.

Syntax:

```
column_name datatype []
```

Above is a one-dimensional array.

To define multiple dimensional array, you add the square brackets.

For example, you can define a two-dimensional array as follows:

```
column_name data_type [][]
```

## Inserting data into an array 

Create a table called `contacts` with the `phones` column defined with an array of text.

```PostgreSQL
CREATE TABLE contacts (
  id SERIAL PRIMARY KEY,
  name VARCHAR (100),
  phones TEXT []
);
```

Insert values into the table:

```PostgreSQL
INSERT INTO contacts (name, phones)
VALUES('John Doe',ARRAY [ '(408)-589-5846','(408)-589-5555' ]);
```

In this example, we use the `ARRAY` constructor to construct an array and insert it into `contacts` table.

Alternatively, you can use curly braces as follows:

```PostgreSQL
INSERT INTO contacts (name, phones)
VALUES('Lily Bush','{"(408)-589-5841"}'),
      ('William Gate','{"(408)-589-5842","(408)-589-58423"}');
```

Notice that when using curly braces, you use single quotes `'` to wrap the array and double-quotes `"` to wrap text array items.

## Querying array Data

The following statement retrieves data from `contacts` table:

```PostgreSQL
SELECT
  name,
  phones
FROM
  contacts;
```

```
name     |              phones
--------------+----------------------------------
 John Doe     | {(408)-589-5846,(408)-589-5555}
 Lily Bush    | {(408)-589-5841}
 William Gate | {(408)-589-5842,(408)-589-58423}
(3 rows)
```

To access an array element, you use the subscript within square brackets `[]`.

By default, PostgreSQL uses one-based numbering for array elements.

The following statement retrieves the contact's name and the first phone number:

```PostgreSQL
SELECT
  name,
  phones [ 1 ]
FROM
  contacts;
```

```
name     |     phones
--------------+----------------
 John Doe     | (408)-589-5846
 Lily Bush    | (408)-589-5841
 William Gate | (408)-589-5842
(3 rows)
```

You can use the array element in the `WHERE` clause as the condition to filter the rows.

For example, the following query finds the contacts who have the phone number `(408)-589-58423` as the second phone number:

```PostgreSQL
SELECT
  name
FROM
  contacts
WHERE
  phones [ 2 ] = '(408)-589-58423';
```

```
name
--------------
 William Gate
(1 row)
```

## Modifying PostgreSQL array 

PostgreSQL allows you to update each element of an array or the whole array.

The following statement updates the second phone number of `ID` $3$.

```PostgreSQL
UPDATE contacts
SET phones [2] = '(408)-589-5843'
WHERE ID = 3
RETURNING *;
```

```
id |     name     |             phones
----+--------------+---------------------------------
  3 | William Gate | {(408)-589-5842,(408)-589-5843}
(1 row)
```

The following statement updates an array as a whole.

```PostgreSQL
UPDATE
  contacts
SET
  phones = '{"(408)-589-5843"}'
WHERE
  id = 3
RETURNING *;
```

```
id |     name     |      phones
----+--------------+------------------
  3 | William Gate | {(408)-589-5843}
(1 row)
```

## Searching in PostgreSQL Array 

Suppose, you want to know who has the phone number `(408)-589-5555` regardless of the position of the phone number in the `phones` array, you can use `ANY()` function as follows:

```PostgreSQL
SELECT
  name,
  phones
FROM
  contacts
WHERE
  '(408)-589-5555' = ANY (phones);
```

```
name   |             phones
----------+---------------------------------
 John Doe | {(408)-589-5846,(408)-589-5555}
(1 row)
```

## Expanding Arrays 

PostgreSQL provides the `unnest()` function to expand an array to a list of rows. For example, the following query expands all phone numbers of the `phones` array.

```PostgreSQL
SELECT
  name,
  unnest(phones)
FROM
  contacts;
```

```
name     |     unnest
--------------+----------------
 John Doe     | (408)-589-5846
 John Doe     | (408)-589-5555
 Lily Bush    | (408)-589-5841
 William Gate | (408)-589-5843
(4 rows)
```

## Summary 

- In PostgreSQL, an array is a collection of elements with the same data type.
- Use the `data_type []` to define a one-dimensional array for a column.
- Use the `[index]` syntax to access the `index` element of an array. The first element has an index of one.
- Use the `unnest()` function to expand an array to a list of rows.