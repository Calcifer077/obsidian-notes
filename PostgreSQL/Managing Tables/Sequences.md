---
title: PostgreSQL Sequences
source: https://neon.com/postgresql/tutorial/sequences
created: 2026-08-04
tags:
  - database
  - postgresql
---
In PostgreSQL, a sequence is a database object that allows you to generate a sequence of unique integers.

Typically, you use a sequence to generate a unique identifier for a primary key in a table. Additionally, you can use a sequence to generate unique numbers across tables.

To create a new sequence, you use the `CREATE SEQUENCE` statement.

Here's the basic syntax:

```PostgreSQL
CREATE SEQUENCE [ IF NOT EXISTS ] sequence_name
    [ AS { SMALLINT | INT | BIGINT } ]
    [ INCREMENT [ BY ] increment ]
    [ MINVALUE minvalue | NO MINVALUE ]
    [ MAXVALUE maxvalue | NO MAXVALUE ]
    [ START [ WITH ] start ]
    [ CACHE cache ]
    [ [ NO ] CYCLE ]
    [ OWNED BY { table_name.column_name | NONE } ]
```

### sequence_name

Specify the name of the sequence after the `CREATE SEQUENCE` clause. The `IF NOT EXISTS` conditionally creates a new sequence only if it does not exist.

The sequence name must be distinct from any other sequences, tables, [indexes](https://neon.com/postgresql/postgresql-indexes), [views](https://neon.com/postgresql/postgresql-views), or foreign tables in the same schema.

### AS { SMALLINT | INT | BIGINT } 

Specify the [data type](https://neon.com/postgresql/tutorial/postgresql-data-types) of the sequence. The valid data type is [`SMALLINT`](https://neon.com/postgresql/tutorial/postgresql-integer), [`INT`](https://neon.com/postgresql/tutorial/postgresql-integer), and [`BIGINT`](https://neon.com/postgresql/tutorial/postgresql-integer). The default data type is `BIGINT` if you skip it.

### INCREMENT [ BY ] increment

The `increment` specifies which value to add to the current sequence value.

A positive number will make an ascending sequence whereas a negative number will form a descending sequence.

The default increment value is 1.

### MINVALUE minvalue | NO MINVALUE,  MAXVALUE maxvalue | NO MAXVALUE

Define the minimum value and maximum value of the sequence. If you use `NO MINVALUE` and `NO MAXVALUE`, the sequence will use the default value.

For an ascending sequence, the default maximum value is the maximum value of the data type of the sequence and the default minimum value is 1.

In the case of a descending sequence, the default maximum value is -1 and the default minimum value is the minimum value of the data type of the sequence.

### START [ WITH ] start

The `START` clause specifies the starting value of the sequence.

The default starting value is `minvalue` for ascending sequences and `maxvalue` for descending ones.

### cache

The `CACHE` determines how many sequence numbers are preallocated and stored in memory for faster access. One value can be generated at a time.

By default, the sequence generates one value at a time i.e., no cache.

### CYCLE | NO CYCLE
The `CYCLE` allows you to restart the value if the limit is reached. The next number will be the minimum value for the ascending sequence and the maximum value for the descending sequence.

If you use `NO CYCLE`, when the limit is reached, attempting to get the next value will result in an error.

The `NO CYCLE` is the default if you don’t explicitly specify `CYCLE` or `NO CYCLE`.

### OWNED BY table_name.column_name

The `OWNED BY` clause allows you to associate the table column with the sequence so that when you drop the column or table, PostgreSQL will automatically drop the associated sequence.

Note that when you use the [`SERIAL`](https://neon.com/postgresql/tutorial/postgresql-serial) pseudo-type for a column of a table, behind the scenes, PostgreSQL automatically creates a sequence associated with the column.

## Examples

### 1) Creating an ascending sequence

This statement uses the `CREATE SEQUENCE` statement to create a new ascending sequence from 100 with an increment of 5:

```PostgreSQL
CREATE SEQUENCE mysequence
INCREMENT 5
START 100;
```

To get the next value from the sequence, you can use `nextval()` function:

```PostgreSQL
SELECT nextval('mysequence');
```

Will print 100.

### 2) Creating a sequence associated with a table column

First, create a new table named `order_details`:

```PostgreSQL
CREATE TABLE order_details(
    order_id SERIAL,
    item_id INT NOT NULL,
    item_text VARCHAR NOT NULL,
    price DEC(10,2) NOT NULL,
    PRIMARY KEY(order_id, item_id)
);
```

Second, create a new sequence associated with the `item_id` column of the created table:

```PostgreSQL
CREATE SEQUENCE order_item_id
START 10
INCREMENT 10
MINVALUE 10
OWNED BY order_details.item_id;
```

Now, if you insert and query the table, table would use a `serial` for `item_id`.

## Listing all sequences in a database

To list all sequences in the current database, you use the following query:

```PostgreSQL
SELECT
    relname sequence_name
FROM
    pg_class
WHERE
    relkind = 'S';
```

## Deleting sequences

If a sequence is associated with a table column, it will be automatically dropped once the table column is removed or the table is dropped.

You can also remove a sequence manually using the `DROP SEQUENCE` statement:

```PostgreSQL
DROP SEQUENCE [ IF EXISTS ] sequence_name [, ...]
[ CASCADE | RESTRICT ];
```

- First, specify the name of the sequence which you want to drop. The `IF EXISTS` option conditionally deletes the sequence if it exists. If you want to drop multiple sequences at once, you can use a list of comma-separated sequence names.
- Then, use the `CASCADE` option if you want to drop objects that depend on the sequence recursively, objects that depend on the dependent objects, and so on.