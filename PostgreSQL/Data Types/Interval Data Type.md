---
title: PostgreSQL Interval Data Type
source: https://neon.com/postgresql/tutorial/interval
created: 2026-10-17
tags:
  - postgresql
---
The interval data type allows you to store and manipulate a period in years, months, days, hours, minutes, and seconds. 

An interval value requires 16 bytes of storage that can store a period with the allowed range from `-178,000,000` years to `178,000,000` years.

Additionally, an interval value can have an optional precision value `p` with the permitted range from $0$ to $6$. The precision `p` is the number of fraction digits retained in the second (time's second) field.

The following examples show some interval values:

```PostgreSQL
interval '2 months ago';
interval '3 hours 20 minutes';
```

Internally, PostgreSQL stores interval values as months, days, and seconds. The months and days values are integers while the seconds field can have fractions.

The interval values are very useful when doing date or time arithmetic. For example, if you want to know the time of 3 hours 20 minutes ago at the current time of the last year, you can use the following statement:

```PostgreSQL
SELECT
	now(),
	now() - INTERVAL '1 year 3 hours 20 minutes'
             AS "3 hours 20 minutes ago of last year";
```

Output:

```
now              | 3 hours 20 minutes ago of last year
-------------------------------+-------------------------------------
 2024-01-31 21:34:52.242914-05 | 2023-01-31 18:14:52.242914-05
(1 row)
```


## Interval Input Format 

PostgreSQL provides you with the following verbose syntax to write the interval values:

```
quantity unit [quantity unit...] [direction]
```

- `quantity` is a number, sign `+` or `-` is also accepted 
- `unit` can be any of millennium, century, decade, year, month, week, day, hour, minute, second, millisecond, or abbreviation (y, m, d, etc.,) or plural forms (months, days, etc.).
- `direction` can be `ago` or empty string `''`

This format is called `postgres_verbose`. The following examples illustrate some interval values that use the verbose syntax:

```PostgreSQL
INTERVAL '1 year 2 months 3 days';
INTERVAL '2 weeks ago';
```

### ISO $8601$ interval format 

In addition to the verbose syntax, PostgreSQL allows you to write the interval values using `ISO 8601` time intervals in two ways: format with designators and alternative format. 

The `ISO 8601` format with designators is like this:

```
P quantity unit [ quantity unit ...] [ T [ quantity unit ...]]
```

In this format, the interval value must start with the letter `P`. The letter `T` is for determining the time-of-day unit.

Following table illustrates the `ISO 8601` interval unit abbreviations:

| **Abbreviation** | **Description**            |
| ---------------- | -------------------------- |
| Y                | Years                      |
| M                | Months (in the date part)  |
| W                | Weeks                      |
| D                | Days                       |
| H                | Hours                      |
| M                | Minutes (in the time part) |
| S                | Seconds                    |

Note that `M` can be months or minutes depending on whether it appears before or after the letter `T`. 

For example, the interval of 6 years 5 months 4 days 3 hours 2 minutes 1 second can be written in the ISO 8601 designators format as follows:

```
P6Y5M4DT3H2M1S
```

The alternative form of `ISO 8601` is:

```
P [ years-months-days ] [ T hours:minutes:seconds ]
```

It must start with the letter `P`, and the letter `T` separates the date and time parts of the interval value.

For example, the interval of `6 years 5 months 4 days 3 hours 2 minutes 1 second` can be written in the `ISO 8601` alternative form as:

```
P0006-05-04T03:02:01
```

## Interval Output Format 

The output style of interval values is set by using the `SET intervalstyle` command, for example:

```PostgreSQL
SET intervalstyle = 'sql_standard';
```

PostgreSQL provides four output formats:

- `sql_standard`
- `postgres` (default)
- `postgresverbose`
- `iso_8601`

The following represents the interval of `6 years 5 months 4 days 3 hours 2 minutes 1 second` in the four styles:

```PostgreSQL
SET intervalstyle = 'sql_standard';
SELECT
  INTERVAL '6 years 5 months 4 days 3 hours 2 minutes 1 second';

SET intervalstyle = 'postgres';
SELECT
  INTERVAL '6 years 5 months 4 days 3 hours 2 minutes 1 second';

SET intervalstyle = 'postgres_verbose';
SELECT
  INTERVAL '6 years 5 months 4 days 3 hours 2 minutes 1 second';

SET intervalstyle = 'iso_8601';
SELECT
  INTERVAL '6 years 5 months 4 days 3 hours 2 minutes 1 second';
```

Output:

| **Format**        | **Output**                                     |
| ----------------- | ---------------------------------------------- |
| `sql_standard`    | `+6-5 +4 +3:02:01`                             |
| `postgres`        | `6 years 5 mons 4 days 03:02:01`               |
| `postgresverbose` | `@ 6 years 5 mons 4 days 3 hours 2 mins 1 sec` |
| `iso_8601`        | `P6Y5M4DT3H2M1S`                               |
## Interval Related operators and functions 

### Interval operators 

You can apply the arithmetic operator (`+`, `-`, `*`, etc.,) to the interval values, for example:

```PostgreSQL
SELECT INTERVAL '2h 50m' + INTERVAL '10m'; -- 03:00:00
SELECT INTERVAL '2h 50m' - INTERVAL '50m'; -- 02:00:00
SELECT 600 * INTERVAL '1 minute'; -- 10:00:00
```

### Converting PostgreSQL interval to string 

To convert an interval value to a string, you use the `TO_CHAR()` function.

```PostgreSQL
TO_CHAR(interval, format)
```

The [`TO_CHAR()`](https://neon.com/postgresql/postgresql-string-functions/postgresql-to_char) function takes the first argument as an interval value, the second one as the format, and returns a string that represents the interval in the specified format.

```PostgreSQL
SELECT
    TO_CHAR(
        INTERVAL '17h 20m 05s',
        'HH24:MI:SS'
    );
```

```
to_char
----------
 17:20:05
(1 row)
```

### Extracting data from a PostgreSQL interval 

To extract fields such as year, month, date, etc., from an interval, you use the [`EXTRACT()`](https://neon.com/postgresql/postgresql-date-functions/postgresql-extract) function.

```PostgreSQL
EXTRACT(field FROM interval)
```

The field can be the year, month, date, hour, minutes, etc., that you want to extract from the interval. The extract function returns a value of type double precision.

```PostgreSQL
SELECT
    EXTRACT (
        MINUTE
        FROM
            INTERVAL '5 hours 21 minutes'
    );
```

```
extract
-----------
        21
(1 row)
```

### Adjusting interval values 

`justify_days` and `justify_hours` allows you to adjust the interval of 30 days as one month and the interval of 24 hours as one day:

```PostgreSQL
SELECT
    justify_days(INTERVAL '30 days'),
    justify_hours(INTERVAL '24 hours');
```

```
justify_days | justify_hours
--------------+---------------
 1 mon        | 1 day
(1 row)
```

In addition, the `justify_interval` function adjusts interval using `justify_days` and `justify_hours` with additional sign adjustments:

```PostgreSQL
SELECT
    justify_interval(interval '1 year -1 hour');
```

```
justify_interval
--------------------------
 11 mons 29 days 23:00:00
(1 row)
```

## Using interval type in a table 

First, create a table called `event` that has three columns `id`, `event_name`, and `duration`. The duration column is `interval` type:

```PostgreSQL
CREATE TABLE event (
    id SERIAL PRIMARY KEY,
    event_name VARCHAR(255) NOT NULL,
    duration INTERVAL NOT NULL
);
```

Second, insert some rows into the `event` table:

```PostgreSQL
INSERT INTO event (event_name, duration)
VALUES
    ('pgConf', '1 hour 30 minutes'),
    ('pgDAY', '2 days 5 hours')
RETURNING *;
```

```
id | event_name | duration
----+------------+----------
  1 | pgConf     | PT1H30M
  2 | pgDAY      | P2DT5H
(2 rows)


INSERT 0 2
```

Now, you can apply any function that we discussed above and more on these intervals. Let's say we want to calculate total duration for all events:

```PostgreSQL
SELECT
    SUM(duration) AS total_duration
FROM event;
```

```
total_duration
----------------
 P2DT6H30M
(1 row)
```

