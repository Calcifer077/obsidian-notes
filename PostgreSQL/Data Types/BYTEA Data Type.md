---
title: PostgreSQL BYTEA Data Type
source: https://neon.com/postgresql/tutorial/bytea-data-type
created: 2026-10-25
tags:
  - postgresql
---
In PostgreSQL, `BYTEA` is a binary data type that you can use to store binary strings or byte sequences/ `BYTEA` stands for binary array.

```PostgreSQL
column_name BYTEA
```

The maximum size of a `BYTEA` column is 1GB. It means you can only store binary data up to 1GB in a single `BYTEA` column. However, storing a large amount of binary data in a `BYTEA` column is not efficient.

If files are larger than a few megabytes, you can store them externally and save the paths to the files in the database.

## Examples 

First, create a table called `binary_data`:

```PostgreSQL
CREATE TABLE binary_data (
	id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
	data BYTEA
)
```

Second, insert a binary string:

```PostgreSQL
INSERT INTO binary_data(data)
VALUES ('\x012345');
```

Third, retrieve data from the `BYTEA` column:

```PostgreSQL
SELECT * FROM binary_data;
```

```
id |   data
----+----------
  1 | \x012345
(1 row)
```

## Summary 

- Use the `BYTEA` data type to store small binary data in the database.