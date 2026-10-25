---
title: PostgreSQL XML Data Type
source: https://neon.com/postgresql/tutorial/xml-data-type
created: 2026-10-25
tags:
  - postgresql
---
PostgreSQL supports built-in `XML` data type that allows you to store XML documents directly within the database.

```PostgreSQL
column_name XML
```

## Examples 

First, create a table:

```PostgreSQL
CREATE TABLE person(
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    info XML
);
```

Second, insert a row:

```PostgreSQL
INSERT INTO person (info)
VALUES (
    XMLPARSE(DOCUMENT '<?xml version="1.0" encoding="UTF-8"?>
    <person>
        <name>John Doe</name>
        <age>35</age>
        <city>San Francisco</city>
    </person>')
);
```

In this statement:

- `DOCUMENT` indicates that the input string is a complete XML document starting with the XML declaration `<?xml version="1.0" encoding="UTF-8"?>` and having the root element `<person>`
- `XMLPARSE` function converts the string into an XML document.
- The `INSERT` statement inserts the new XML document into the info column of the `person` table.

Insert multiple rows:

```PostgreSQL
INSERT INTO person (info)
VALUES
(
    XMLPARSE(DOCUMENT '<?xml version="1.0" encoding="UTF-8"?>
    <person>
        <name>Jane Doe</name>
        <age>30</age>
        <city>San Francisco</city>
    </person>')
),
(
    XMLPARSE(DOCUMENT '<?xml version="1.0" encoding="UTF-8"?>
    <person>
        <name>John Smith</name>
        <age>40</age>
        <city>New York</city>
    </person>')
),
(
    XMLPARSE(DOCUMENT '<?xml version="1.0" encoding="UTF-8"?>
    <person>
        <name>Alice Johnson</name>
        <age>30</age>
        <city>Los Angeles</city>
    </person>')
);
```

Retrieve the names of persons from the XML documents using `xpath()` function:

```PostgreSQL
SELECT xpath('/person/name/text()', info) AS name 
FROM person;
```

```
name
-------------------
 {"John Doe"}
 {"Jane Doe"}
 {"John Smith"}
 {"Alice Johnson"}
(4 rows)
```

Each row in the result set is an array of XML values representing person names. Since each person has one name, the result array has only one element.

Retrieve person names as text from the XML documents using `xpath()` function:

```PostgreSQL
SELECT (xpath('/person/name/text()', info))[1]::text AS name
FROM person;
```

```
name
---------------
 John Doe
 Jane Doe
 John Smith
 Alice Johnson
(4 rows)
```

How it works.

- First, the XPath `'/person/name/text()'` returns the text of the name node of the XML document. It returns an array that includes all matching values.
- Second, the `[1]` subscript returns the first element of the array.
- Third, the `::text` casts the XML value to the text.

Retrieve the name, age, and city from the XML document:

```PostgreSQL
SELECT
    (xpath('/person/name/text()', info))[1]::text AS name,
    (xpath('/person/age/text()', info))[1]::text::integer AS age,
    (xpath('/person/city/text()', info))[1]::text AS city
FROM
    person;
```

```
name      | age |     city
---------------+-----+---------------
 John Doe      |  35 | San Francisco
 Jane Doe      |  30 | San Francisco
 John Smith    |  40 | New York
 Alice Johnson |  30 | Los Angeles
(4 rows)
```

## Creating indexes for XML data

If the person table has many rows, finding the person by name will be slow. You can create an expression index for the XML documents to improve the query performance.

```PostgreSQL
CREATE INDEX person_name
ON person USING BTREE
    (cast(xpath('/person/name', info) as text[])) ;
```

>You can see the testing of the above query on the source website.

## Summary 

- Use the `XML` data type to store XML documents in the database.
- Use the `xpath()` function to retrieve a value from XML documents.