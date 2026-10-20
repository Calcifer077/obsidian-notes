---
title: PostgreSQL Timestamp Data Type
source: https://neon.com/postgresql/tutorial/timestamp
created: 2026-10-17
tags:
  - postgresql
---
PostgreSQL provides you with two temporal data types for handling timestamps:

- `timestamp`: a timestamp without a timezone.
- `timestamptz`: timestamp with a timezone.

The `timestamp` datatype allows you to store both [date](https://neon.com/postgresql/tutorial/postgresql-date) and time. However, it does not have any time zone data. It means that when you change the time zone of your database server, the timestamp value stored in the database will not change automatically.

The `timestamptz` datatype is the timestamp with a time zone. The `timestamptz` data type is a time zone-aware date and time data type.

Internally, PostgreSQL stores the `timestamptz` in UTC value.

- When you insert a value into a `timestamptz` column, PostgreSQL converts the `timestamptz` value into a UTC value and stores the UTC value in the table.
- When you retrieve data from a `timestamptz` column, PostgreSQL converts the UTC value back to the time value of the time zone set by the database server, the user, or the current database connection.

Both `timestamp` and `timestamptz` uses 8 bytes for storing values.

It’s important to note that PostgreSQL stores `timestamptz` values in the database using UTC values. It does not store any timezone data with the `timestamptz` value.

## PostgreSQL timestamp example 

First, create a table that consists of both `timestamp` and `timestamptz` columns.

```PostgreSQL
CREATE TABLE timestamp_demo (
    ts TIMESTAMP,
    tstz TIMESTAMPTZ
);
```

Next, set the timezone of the database server to `America/Los_Angeles`.

You can see the current time zone using the `SHOW TIMEZONE` command.

Insert a row into the `timestamp_demo` table:

```PostgreSQL
INSERT INTO timestamp_demo (ts, tstz)
VALUES('2016-06-22 19:10:25-07','2016-06-22 19:10:25-07');
```

If you query the table, you would get both values (`timestamp` and `timestamptz`) as same:

```PostgreSQL
SELECT
   ts, tstz
FROM
   timestamp_demo;
```

```
ts          |          tstz
---------------------+------------------------
 2016-06-22 19:10:25 | 2016-06-22 19:10:25-07
(1 row)
```

Now, if you change the timezone if your server, and query the same thing:

```PostgreSQL
SET timezone = 'America/New_York';
```

```PostgreSQL
SELECT
  ts,
  tstz
FROM
  timestamp_demo;
```

```
ts          |          tstz
---------------------+------------------------
 2016-06-22 19:10:25 | 2016-06-22 22:10:25-04
(1 row)
```

The value in the `timestamp` column does not change whereas the value in the `timestamptz` column is adjusted to the new time zone of `'America/New_York'`.

Generally, it is a good practice to use the `timestamptz` data type to store the timestamp data.

## PostgreSQL timestamp functions 

### Getting the current time 

To get the current timestamp you use the `NOW()` function as follows:

```PostgreSQL
SELECT NOW();
```

```
now
-------------------------------
 2024-01-31 21:01:58.985943-05
(1 row)
```

Alternatively, you can use the `CURRENT_TIMESTAMP` function:

```PostgreSQL
SELECT CURRENT_TIMESTAMP;
```

To get the current time without a date, you use the `CURRENT_TIME` function:

```PostgreSQL
SELECT CURRENT_TIME;
```

Note that both `CURRENT_TIMESTAMP` and `CURRENT_TIME` return the current time with the timezone. 

To get the time of day in the string format, you use the `timeofday()` function.

```PostgreSQL
SELECT TIMEOFDAY();
```

```
timeofday
-------------------------------------
 Wed Jan 31 21:02:20.840159 2024 EST
(1 row)
```

### Convert between time zones 

To convert a timestamp to another time zone, you use the `timezone(zone, timestamp)` function.

We set the current time zone to `America/New_York`. To convert `2016-06-01 00:00` to `America/Los_Angeles` time zone, you use the `timezone()` function as follows:

```PostgreSQL
SELECT timezone('America/Los_Angeles','2016-06-01 00:00');
```

```
timezone
---------------------
 2016-05-31 21:00:00
(1 row)
```

## Using default values for timestamp columns 

First, create a new table called `department`:

```PostgreSQL
CREATE TABLE department (
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL,
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);
```

The default values for the `created_at` and `updated_at` columns are the current timestamp provided by the `CURRENT_TIMESTAMP` function. 

If you insert values into `department` table:

```PostgreSQL
INSERT INTO department(name)
VALUES('IT')
RETURNING *;
```

Output:

```
id | name |          created_at           |          updated_at
----+------+-------------------------------+-------------------------------
  1 | IT   | 2024-01-31 21:25:31.162808-05 | 2024-01-31 21:25:31.162808-05
(1 row)
```

When you update a row in the `department` table, the `updated_at` column will not be updated to the current time automatically.

To update the value in the `updated_at` column at the time the row is updated, you can create a `BEFORE UPDATE` trigger to change the value in the `updated_at` column.

```PostgreSQL
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = current_timestamp;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER department_updated_at_trigger
BEFORE UPDATE ON department
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();
```

> Will add a link here for triggers, when I will learn about them.

Now, whenever you update any row or column in `department` table, `updated_at` will also update.

## Summary 

- Use `timestamp` and `timestamptz` to store timestamp data.
- PostgreSQL stores the `timestamptz` values in the database as UTC values.

