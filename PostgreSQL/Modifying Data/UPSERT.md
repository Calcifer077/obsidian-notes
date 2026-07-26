---
title: PostgreSQL UPSERT using INSERT ON CONFLICT Statement
source: https://neon.com/postgresql/tutorial/upsert
created: 2026-07-25
tags:
  - database
  - postgresql
---
Upsert is a combination of [`UPDATE`](UPDATE.md) and [`INSERT`](INSERT.md). It updates the record if it already exists, or creates a new record if it does not.

PostgreSQL does not have the `UPSERT` statement but it supports the upsert operation via the `INSERT...ON CONFLICT` statement.

_Note: Versions over 15 can use [`MERGE`](MERGE.md) statement which is equivalent to the `UPSERT` statement._

Basic syntax of `INSERT...ON CONFLICT`:

```PostgreSQL
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...)
ON CONFLICT (conflict_column)
DO NOTHING | DO UPDATE SET column1 = value1, column2 = value2, ...;
```

In this syntax:

- `table_name`: This is the name of the table into which you want to insert data.
- `(column1, column2, ...)`: The list of columns you want to insert values into the table.
- `VALUES(value1, value2, ...)`: The values you want to insert into the specified columns `(column1, column2, ...)`.
- `ON CONFLICT (conflict_column):` This clause specifies the conflict target, which is the [unique constraint](https://neon.com/postgresql/tutorial/postgresql-unique-constraint) or [unique index](https://neon.com/postgresql/postgresql-indexes/postgresql-unique-index) that may cause a conflict. `conflict_column` is that column on which conflict can occur.
- `DO NOTHING`: This instructs PostgreSQL to do nothing when a conflict occurs. It won't insert the new row.
- `DO UPDATE`: This performs an update if a conflict occurs.
- `SET column = value1, column = value2, ...`: This list of the columns to be updated and their corresponding values in case of conflict.

How the `INSERT...ON CONFLICT` statement works:

- First, the `ON CONFLICT` clause identifies the conflict target which is usually a unique constraint (or a unique index). If the data that we insert violates the constraint, a conflict occurs.
- Second, the `DO UPDATE` instructs PostgreSQL to update existing rows or do nothing rather than abording the entire operation when a conflict occurs.
- Third, the `SET` clause defines the columns and values to update. You can use new values or reference the values you attempted to insert using the `EXCLUDED` keyword.

## Examples

The following statement create the `inventory` table and insert data into it:

```PostgreSQL
CREATE TABLE inventory(
   id INT PRIMARY KEY,
   name VARCHAR(255) NOT NULL,
   price DECIMAL(10,2) NOT NULL,
   quantity INT NOT NULL
);

INSERT INTO inventory(id, name, price, quantity)
VALUES
	(1, 'A', 15.99, 100),
	(2, 'B', 25.49, 50),
	(3, 'C', 19.95, 75)
RETURNING *;
```

The `inventory` table contains information about various products, including their names, prices, and quantities in stock.

Suppose we've received an updated list of products with new prices, and now we need to update the inventory accordingly. 

In this case, the upsert operation can be handy to handle the following situations:

- **Updating existing products**: If a product already exists in the `inventory` table and we want to update its price and quantity with the new information.
- **Insert new products**: If a product is not in `inventory` table and we want to insert it into the table.

### 1) Basic example

The following example uses the `INSERT...ON CONFLICT` statement to insert a new row into the `inventory` table:

```PostgreSQL
INSERT INTO inventory (id, name, price, quantity)
VALUES (1, 'A', 16.99, 120)
ON CONFLICT(id)
DO UPDATE SET
  price = EXCLUDED.price,
  quantity = EXCLUDED.quantity;
```

In this example, we attempted to insert a new row into the `inventory` table. However, the `inventory` table already has a row with `id` `1` (primary key), therefore, a conflict occurs. 

The `DO UPDATE` changes the price and quantity of the product to the new value being inserted, rest remains the same. The `EXCLUDED` allows us to access the new values.

_Note: if there was no conflict, the new row would have just been inserted as it is, like in the below example. _

```PostgreSQL
INSERT INTO inventory (id, name, price, quantity)
VALUES (4, 'D', 29.99, 20)
ON CONFLICT(id)
DO UPDATE SET
  price = EXCLUDED.price,
  quantity = EXCLUDED.quantity;
```

As there is no `id` of `4`, so there would be no conflict and insertion works fine.

## Summary

- Use the PostgreSQL upsert to update data if it already exists or insert the data if it does not.
- Use the `INSERT...ON CONFLICT` statement for upsert.