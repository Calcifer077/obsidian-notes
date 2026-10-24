---
title: PostgreSQL JSON
source: https://neon.com/postgresql/tutorial/json
created: 2026-10-24
tags:
  - postgresql
---
PostgreSQL offers two data types for storing JSON:

- JSON - store an exact copy of the JSON text.
- JSONB - store the JSON data in binary format.

## JSON vs. JSONB 

|Feature|JSON|JSONB|
|---|---|---|
|Storage|Textual representation (verbatim)|Binary storage format|
|Size|Typically larger because it retains the whitespace in JSON data|Typically smaller|
|Indexing|Full-text search indexes|Binary indexes|
|Performance|Slightly slower|Generally faster|
|Query performance|Slower due to parsing|Faster due to binary storage|
|Parsing|Parse each time|Parse once, store in binary format|
|Data manipulation|Simple and easy|More complex|
|Ordering of keys|Preserved|Not preserved|
|Duplicate keys|Allow duplicate key, the last value is retained|Do not allow duplicate keys.|
|Use cases|Storing configuration data, log data, simple JSON documents|Storing JSON documents where fast querying and indexing are required|

In practice, you should use JSONB to store JSON data unless you have specialized requirements such as retaining the ordering of keys in the JSON documents.

## Examples 

### 1) Storing JSON objects 

First, create a table:

```PostgreSQL
CREATE TABLE products(
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    properties JSONB
);
```

Second, insert JSON data into it:

```PostgreSQL
INSERT INTO products(name, properties)
VALUES('Ink Fusion T-Shirt','{"color": "white", "size": ["S","M","L","XL"]}'),
	('ThreadVerse T-Shirt','{"color": "black", "size": ["S","M","L","XL"]}'),
      ('Design Dynamo T-Shirt','{"color": "blue", "size": ["S","M","L","XL"]}')
RETURNING *;
```

Third, retrieve JSON data from the `products` table:

```PostgreSQL
SELECT id, name, properties
FROM products;
```

```
id |         name          |                    properties
----+-----------------------+---------------------------------------------------
  1 | Ink Fusion T-Shirt    | {"size": ["S", "M", "L", "XL"], "color": "white"}
  2 | ThreadVerse T-Shirt   | {"size": ["S", "M", "L", "XL"], "color": "black"}
  3 | Design Dynamo T-Shirt | {"size": ["S", "M", "L", "XL"], "color": "blue"}
(3 rows)
```

Fourth, retrieve the product with the colors extracted from the JSON data in the `properties` column:

```PostgreSQL
SELECT
  id,
  name,
  properties -> 'color' color
FROM
  products;
```

```
id |         name          |  color
----+-----------------------+---------
  1 | Ink Fusion T-Shirt    | "white"
  2 | ThreadVerse T-Shirt   | "black"
  3 | Design Dynamo T-Shirt | "blue"
(3 rows)
```

>The `color` outside of semicolon is alias. It can be anything. 

The `->` operator extracts a JSON object field by a key. There is a sibling operator of `->` which goes by `->>` that extracts strings. 

Above code output shows `color` in inverted commas, but `->>` removes those inverted commas, so that you can do casting, comparison, filtering etc.

```PostgreSQL
SELECT
  id,
  name,
  properties ->> 'color' color
FROM
  products;
```

```
id |         name          | color
----+-----------------------+-------
  1 | Ink Fusion T-Shirt    | white
  2 | ThreadVerse T-Shirt   | black
  3 | Design Dynamo T-Shirt | blue
(3 rows)
```

You can apply filtering on the above result, say you only want colors black and white.

```PostgreSQL
SELECT
  id,
  name,
  properties ->> 'color' color
FROM
  products
WHERE
  properties ->> 'color' IN ('black', 'white');
```

```
id |        name         | color
----+---------------------+-------
  1 | Ink Fusion T-Shirt  | white
  2 | ThreadVerse T-Shirt | black
(2 rows)
```

### 2) Storing JSON arrays 

First, create a table called `contacts` for storing the contact persons:

```PostgreSQL
CREATE TABLE contacts(
   id SERIAL PRIMARY KEY,
   name VARCHAR(255) NOT NULL,
   phones JSONB
);
```

The `phones` column has the JSONB data type. We’ll store both work phone and personal phone numbers in a JSON array in the `phones` column.

Second, insert new rows into the `contacts` table:

```PostgreSQL
INSERT INTO contacts(name, phones)
VALUES
   ('John Doe','["408-111-2222", "408-111-2223"]'),
   ('Jane Doe','["212-111-2222", "212-111-2223"]')
RETURNING *;
```

```
id |   name   |              phones
----+----------+----------------------------------
  1 | John Doe | ["408-111-2222", "408-111-2223"]
  2 | Jane Doe | ["212-111-2222", "212-111-2223"]
(2 rows)
```

Third, retrieve the contacts with the work phone numbers from the `contacts` table:

```PostgreSQL
SELECT
  name,
  phones ->> 0 "work phone"
FROM
  contacts;
```

```
name   |  work phone
----------+--------------
 John Doe | 408-111-2222
 Jane Doe | 212-111-2222
(2 rows)
```

The `->> index` extracts the index element in an array. In this example, we use the `->> 0` to extract the first elements in the `phones` array as text.

[Check JSON functions to process JSON data](https://neon.com/postgresql/postgresql-json-functions).

## Summary 

- JSON stands for JavaScript Object Notation.
- Use JSONB data type to store JSON data.