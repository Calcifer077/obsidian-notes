---
title: PostgreSQL UPDATE join
source: https://neon.com/postgresql/tutorial/update-join
created: 2026-07-24
tags:
  - database
  - postgresql
---
Sometimes, we need to update data in a table based on values in another table. In this case, we can use [`UPDATE`](UPDATE.md) join.

Basic syntax:

```PostgresQL
UPDATE table1
SET table1.c1 = new_value
FROM table2
WHERE table1.c2 = table2.c2;
```

To update a table with conditions from another table, we specify the joined table (table2) in the `FROM` clause and provide the join condition in the `WHERE` clause. The `FROM` clause must appear immediately after the `SET` clause.

For each row of the table `table1`, the `UPDATE` statement examines every row of the table `table2`.

If the values in the `c2` column of table `table1` equals the values in the `c2` column of table `table2`, the `UPDATE` statement updates the value in the `c1` column of the table `table1` with the new value (`new_value`).

## Examples

We will use the following database tables for demonstration.

![](../../assets/PostgreSQL-UPDATE-JOIN-Sample-Database.avif)

Let's create tables and insert data into them.

```PostgreSQL
CREATE TABLE product_segment (
    id SERIAL PRIMARY KEY,
    segment VARCHAR NOT NULL,
    discount NUMERIC (4, 2)
);


INSERT INTO
    product_segment (segment, discount)
VALUES
    ('Grand Luxury', 0.05),
    ('Luxury', 0.06),
    ('Mass', 0.1);
    
CREATE TABLE product(
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL,
    price NUMERIC(10,2),
    net_price NUMERIC(10,2),
    segment_id INT NOT NULL,
    FOREIGN KEY(segment_id) REFERENCES product_segment(id)
);


INSERT INTO
    product (name, price, segment_id)
VALUES
    ('diam', 804.89, 1),
    ('vestibulum aliquet', 228.55, 3),
    ('lacinia erat', 366.45, 2),
    ('scelerisque quam turpis', 145.33, 3),
    ('justo lacinia', 551.77, 2),
    ('ultrices mattis odio', 261.58, 3),
    ('hendrerit', 519.62, 2),
    ('in hac habitasse', 843.31, 1),
    ('orci eget orci', 254.18, 3),
    ('pellentesque', 427.78, 2),
    ('sit amet nunc', 936.29, 1),
    ('sed vestibulum', 910.34, 1),
    ('turpis eget', 208.33, 3),
    ('cursus vestibulum', 985.45, 1),
    ('orci nullam', 841.26, 1),
    ('est quam pharetra', 896.38, 1),
    ('posuere', 575.74, 2),
    ('ligula', 530.64, 2),
    ('convallis', 892.43, 1),
    ('nulla elit ac', 161.71, 3);
```

Suppose we have to calculate the net price of every product based on the discount of the product segment. To do this, we can apply the `UPDATE` join statement as follows:

```PostgreSQL
UPDATE product
SET net_price = price - price * discount
FROM product_segment
WHERE product.segment_id = product_segment.id;
```

Above statement joins the `product` table to the `product_segment` table. If there is a match in both tables, it gets a discount from the `product_segment` table, calculates the net price based on the following formula, and updates the `net_price` column.

```
net_price = price - price * discount;
```

## Summary

- Use the PostgreSQL `UPDATE` join statement to update data in a table based on values in another table.