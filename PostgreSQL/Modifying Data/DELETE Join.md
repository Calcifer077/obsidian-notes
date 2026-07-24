---
title: PostgreSQL DELETE Join
source: https://neon.com/postgresql/tutorial/delete-join
created: 2026-07-24
tags:
  - database
  - postgresql
---
PostgreSQL does not support the [DELETE JOIN statement like MySQL](https://www.mysqltutorial.org/mysql-basics/mysql-delete-join/). Instead, it offers the `USING` clause in the  [`DELETE`](DELETE.md) statement that provides similar functionality to the `DELETE JOIN`.

Basic syntax:

```PostgreSQL
DELETE FROM table1
USING table2
WHERE condition
RETURNING returning_columns;
```

In this syntax:

- First, specify the name of the table (`table1`) from which you want to delete data after the `DELETE FROM` keywords
- Second, provide a table (`table2`) to join with the main table after the `USING` keyword.
- Third, define a condition in the `WHERE` clause for joining two tables.
- Finally, return the deleted rows in the `RETURNING` clause. The `RETURNING` clause is optional.

## Examples

We will use some example tables for these examples.

```PostgreSQL
CREATE TABLE member(
   id SERIAL PRIMARY KEY,
   first_name VARCHAR(50) NOT NULL,
   last_name VARCHAR(50) NOT NULL,
   phone VARCHAR(15) NOT NULL
);


CREATE TABLE denylist(
    phone VARCHAR(15) PRIMARY KEY
);


INSERT INTO member(first_name, last_name, phone)
VALUES ('John','Doe','(408)-523-9874'),
       ('Jane','Doe','(408)-511-9876'),
       ('Lily','Bush','(408)-124-9221');


INSERT INTO denylist(phone)
VALUES ('(408)-523-9874'),
       ('(408)-511-9876');
```

### 1) Basic example

The following statement deletes rows in the `member` table if the phone number exists in the `denylist` table:

```PostgreSQL
DELETE FROM member
USING denylist
WHERE member.phone = denylist.phone;
```

### 2) Delete join using a subquery example

The `USING` clause is not a part of the SQL standard, meaning that it may not be available in other database systems.

If we intend to ensure compatibility with various database products, we should avoid using the `USING` clause. Instead, we may use a subquery.

Same thing as the previous example just with a subquery:

```PostgreSQL
DELETE FROM member
WHERE phone IN (
    SELECT
      phone
    FROM
      denylist
);
```

In this example:

- First, the subquery returns a list of phones from the `denylist` table.
- Second, the `DELETE` statement deletes rows in the member table whose values in the phone column are in the list of phones returned by the subquery.

## Summary

- Use the `DELETE USING` statement or a subquery to emulate the `DELETE JOIN` operation.