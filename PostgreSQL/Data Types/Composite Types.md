---
title: PostgreSQL Composite Types
source: https://neon.com/postgresql/tutorial/composite-types
created: 2026-10-25
tags:
  - postgresql
---
To define a composite type, use the `CREATE TYPE` statement followed by the type name and a list of fields with their corresponding data types.

```PostgreSQL
CREATE TYPE type_name AS (
    field1 data_type1,
    field2 data_type2,
    ...
);
```

For example, the following statement defines the address type that stores address information including street, city, state, zip code, and country:

```PostgreSQL
CREATE TYPE address_type AS(
   street text,
   city text,
   state text,
   zip_code integer,
   country text
);
```

After defining a composite type, you use it as a data type of a table column.

For example, the following statement creates a table called `contacts` whose type of `address` column is the `address_type`:

```PostgreSQL
CREATE TABLE contacts(
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    address address_type
);
```

Note that when creating a table, PostgreSQL implicitly creates a corresponding composite type. In this example, PostgreSQL automatically creates the `contacts` composite type.

## Inserting values into a composite column 

To construct a composite value, you use the `ROW` expression syntax:

```PostgreSQL
ROW(value1, value2, ...)
```

In this syntax, `ROW` is optional. 

To indicate a `NULL`, you can use the `NULL` keyword:

```PostgreSQL
(value1, NULL, value3, ...)
```

Alternatively, you can omit it in the expression:

```PostgreSQL
(value1,, value3, ...)
```

For example, the following inserts a new row into the `contacts` table:

```PostgreSQL
INSERT INTO contacts (name, address)
VALUES (
    'John Smith',
    ROW('123 St', 'Houston', 'TX', 77001, 'USA')
);
```

The following statement inserts a value into individual fields of the `address` column:

```PostgreSQL
INSERT INTO contacts (
    name,
    address.street,
    address.city,
    address.state,
    address.zip_code
  )
VALUES
  ('Jane Doe', '4000 N. 1st Street', 'San Jose', 'CA', 95134);
```

## Querying composite values 

The following statement retrieves the rows from the `contacts` table:

```PostgreSQL
SELECT * FROM contacts;
```

```
id |    name    |                   address
----+------------+---------------------------------------------
  1 | John Smith | ("123 St",Houston,TX,77001,USA)
  2 | Jane Doe   | ("4000 N. 1st Street","San Jose",CA,95134,)
(2 rows)
```

To query individual fields of a composite type, you use the following syntax:

```
(column_name).field_name
```

```PostgreSQL
SELECT
  id,
  name,
  (address).city,
  (address).state,
  (address).zip_code
FROM
  contacts;
```

```
id |    name    |   city   | state | zip_code
----+------------+----------+-------+----------
  1 | John Smith | Houston  | TX    |    77001
  2 | Jane Doe   | San Jose | CA    |    95134
(2 rows)
```

If you retrieve all fields from a composite value, you can use the asterisk (`*`) shorthand:

```PostgreSQL
SELECT
  id,
  name,
  (address).*
FROM
  contacts;
```

```
id |    name    |       street       |   city   | state | zip_code | country
----+------------+--------------------+----------+-------+----------+---------
  1 | John Smith | 123 St             | Houston  | TX    |    77001 | USA
  2 | Jane Doe   | 4000 N. 1st Street | San Jose | CA    |    95134 | null
(2 rows)
```

## Updating composite values 

The following example updates the country of the contact id 2 to `USA`:

```PostgreSQL
UPDATE contacts
SET
  address.country = 'USA'
WHERE
  id = 2;
```

## Summary 

- Composite types allow you to define custom data types that include multiple fields.