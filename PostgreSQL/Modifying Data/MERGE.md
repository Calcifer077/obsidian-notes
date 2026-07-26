---
title: PostgreSQL MERGE statement
source: https://neon.com/postgresql/tutorial/merge
created: 2026-07-25
tags:
  - database
  - postgresql
---
Think of `MERGE` as a smart helper that can look at our data and decide whether to add new records, update existing ones, or even delete records, all in a single command.

## Basic Concepts

Before we dive into `MERGE`, let's understand some basic terms:

- **Target Table**: The table that we want to modify
- **Source Table**: The table containing our new or updated Data
- **Match Condition**: The rule that determines if records match between our tables.

## Basic Syntax

```PostgreSQL
MERGE INTO target_table
USING source_table
ON match_condition
WHEN MATCHED AND condition THEN
    UPDATE SET column1 = value1, column2 = value2
WHEN MATCHED AND NOT condition THEN
    DELETE
WHEN NOT MATCHED THEN
    INSERT (column1, column2) VALUES (value1, value2)
RETURNING merge_action(), target_table.*;
```

This `MERGE` statement performs three conditional actions on `target_table` based on rows from `source_table`:

1. **Update rows**: If a match is found (`ON match_condition`) and `condition` is true, it updates `column1` and `column2` in `target_table`.
2. **Delete rows**: If a match is found but `condition` is false, it deletes the matching rows in `target_table`.
3. **Insert rows**: If no match is found, it inserts new rows into `target_table` using values from `source_table`.
4. The `RETURNING` clause provides details of the operation (`merge_action()`) and the affected rows.

## Key features in PostgreSQL 17

The new RETURNING clause support in PostgreSQL 17 offers several advantages:

1. **Action Tracking**: The `merge_action()` function tells you exactly what happened to each row
2. **Complete Row Access**: You can return both old and new values for affected rows
3. **Immediate Feedback**: No need for separate queries to verify the results

## Examples

Let's first setup sample tables:

```PostgreSQL
-- Create the main products table
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name TEXT UNIQUE,
    price DECIMAL(10,2),
    stock INTEGER,
    status TEXT,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert some initial data
INSERT INTO products (name, price, stock, status) VALUES
    ('Laptop', 999.99, 50, 'active'),
    ('Keyboard', 79.99, 100, 'active'),
    ('Mouse', 29.99, 200, 'active');

-- Create a table for our updates
CREATE TABLE product_updates (
    name TEXT,
    price DECIMAL(10,2),
    stock INTEGER,
    status TEXT
);

-- Insert mixed update data (new products, updates, and discontinuations)
INSERT INTO product_updates VALUES
    ('Laptop', 1099.99, 75, 'active'),      -- Update: price and stock change
    ('Monitor', 299.99, 30, 'active'),      -- Insert: new product
    ('Keyboard', NULL, 0, 'discontinued'),  -- Delete: mark as discontinued
    ('Headphones', 89.99, 50, 'active');    -- Insert: another new product
```

### 1) Using `MERGE` with `RETURNING`

Now let's see how `MERGE` command can handle all three operations (`INSERT`, `UPDATE`, `DELETE`) while providing detailed feedback through the `RETURNING` clause:

```PostgreSQL
MERGE INTO products p
USING product_updates u
ON p.name = u.name
WHEN MATCHED AND u.status = 'discontinued' THEN
    DELETE
WHEN MATCHED AND u.status = 'active' THEN
    UPDATE SET
        price = COALESCE(u.price, p.price),
        stock = u.stock,
        status = u.status,
        last_updated = CURRENT_TIMESTAMP
WHEN NOT MATCHED AND u.status = 'active' THEN
    INSERT (name, price, stock, status)
    VALUES (u.name, u.price, u.stock, u.status)
RETURNING
    merge_action() as action,
    p.product_id,
    p.name,
    p.price,
    p.stock,
    p.status,
    p.last_updated;
```

_Note: `COALESCE` function evaluates a list of arguments from left to right and returns the first non-null values._

#### Understanding the output

The `RETURNING` clause will provide detailed information about each operation:

```
action  | product_id |    name    |  price   | stock |   status    |      last_updated
---------+------------+------------+----------+-------+-------------+------------------------
 UPDATE  |     1      | Laptop     | 1099.99  |   75  | active      | 2024-12-04 17:41:58.226807
 INSERT  |     4      | Monitor    |  299.99  |   30  | active      | 2024-12-04 17:41:58.226807
 DELETE  |     2      | Keyboard   |   79.99  |  100  | active      | 2024-12-04 17:41:47.816064
 INSERT  |     5      | Headphones |   89.99  |   50  | active      | 2024-12-04 17:41:58.226807
```

Let's break down what happened:

1. **`UPDATE`**: The Laptop's price and stock were updated
2. **`DELETE`**: The Keyboard is deleted from the products table
3. **`INSERT`**: New Monitor and Headphones products were added

### Advanced Usage with Conditions

We can add more complex conditions to our `MERGE` statement:

```PostgreSQL
MERGE INTO products p
USING (
    SELECT
        name,
        price,
        stock,
        status,
        CASE
            WHEN price IS NULL AND status = 'discontinued' THEN 'DELETE'
            WHEN stock = 0 THEN 'OUT_OF_STOCK'
            ELSE status
        END as action_type
    FROM product_updates
) u
ON p.name = u.name
WHEN MATCHED AND u.action_type = 'DELETE' THEN
    DELETE
WHEN MATCHED AND u.action_type = 'OUT_OF_STOCK' THEN
    UPDATE SET
        status = 'inactive',
        stock = 0,
        last_updated = CURRENT_TIMESTAMP
WHEN MATCHED THEN
    UPDATE SET
        price = COALESCE(u.price, p.price),
        stock = u.stock,
        status = u.status,
        last_updated = CURRENT_TIMESTAMP
WHEN NOT MATCHED AND u.action_type != 'DELETE' THEN
    INSERT (name, price, stock, status)
    VALUES (u.name, u.price, u.stock, u.status)
RETURNING
    merge_action() as action,
    p.*,
    u.action_type;
```

## Summary

- Use PostgreSQL's `MERGE` when you are not sure whether to insert new records or update existing ones. `MERGE` gives a single place for updating, inserting and even deleting based on some condition.



