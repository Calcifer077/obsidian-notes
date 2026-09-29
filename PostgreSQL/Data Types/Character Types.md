---
title: "PostgreSQL Character Types: CHAR, VARCHAR, and TEXT"
source: https://neon.com/postgresql/tutorial/char-varchar-text
created: 2026-09-29
tags:
  - postgresql
---
PostgreSQL provides three primary character types:

- `CHARACTER(n)` or `CHAR(n)`
- `CHARACTER VARYING(n)` or `VARCHAR(n)`
- `TEXT`

In this syntax, `n` is a positive integer that specifies the number of characters.

|**Character Types**|**Description**|
|---|---|
|`CHARACTER VARYING(n)`, `VARCHAR(n)`|variable-length with length limit|
|`CHARACTER(n)`, `CHAR(n)`|fixed-length, blank padded|
|`TEXT`, `VARCHAR`|variable unlimited length|
Both `CHAR(n)` and `VARCHAR(n)` can store up to `n` characters. If you attempt to store a string that has more than `n` characters, PostgreSQL will issue an error. However, if the excessive characters are all spaces, PostgreSQL truncates the spaces to the maximum length (`n`) and stores the trimmed characters.

If a string explicitly [casts](https://neon.com/postgresql/tutorial/postgresql-cast) to a `CHAR(n)` or `VARCHAR(n)`, PostgreSQL will truncate the string to `n` characters before inserting it into the table.

The `TEXT` data type can store a string with unlimited length.

If you do not specify the `n` integer for the `VARCHAR` data type, it behaves like the `TEXT` datatype. 

The advantage of specifying the length specifier for the `VARCHAR` data type is that PostgreSQL will issue an error if you attempt to insert a string that has more than `n` characters into the `VARCHAR(n)` column.

Unlike `VARCHAR`, The `CHARACTER` or `CHAR` without the length, specifier (`n`) is the same as the `CHARACTER(1)` or `CHAR(1)`.

Different from other database systems, in PostgreSQL, there is no performance difference among the three character types.

In most cases, you should use `TEXT` or `VARCHAR` and use the `VARCHAR(n)` only when you want PostgreSQL to check the length.