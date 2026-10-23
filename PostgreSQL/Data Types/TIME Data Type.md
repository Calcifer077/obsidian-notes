---
title: PostgreSQL TIME Data Type
source: https://neon.com/postgresql/tutorial/time
created: 2026-10-23
tags:
  - postgresql
---
PostgreSQL provides the `TIME` data type that allows you to store the time data in the database. 

Here's the syntax for declaring a column with the `TIME` data type:

```PostgreSQL
column_name TIME(precision);
```

In this syntax, the `precision` specifies the fractional seconds precision for the time value, which ranges from 1 to 6. 

The `TIME` data type requires 8 bytes and its allowed range is from `00:00:00` to `24:00:00`.

The following illustrates the common formats of the `TIME` values:

```
HH:MI
HH:MI:SS
HHMISS
```

Example:

```
01:02
01:02:03
010203
```

To use a time value with the precision, you can utilize the following formats:

```
MI:SS.pppppp
HH:MI:SS.pppppp
HHMISS.pppppp
```

In this syntax, `p` specifies the precision. For example:

```
04:59.999999
04:05:06.777777
040506.777777
```

`p` is how many digits after the decimal point of the seconds you keep:

- `p = 0` → whole seconds
- `p = 3` → milliseconds
- `p = 6` → microseconds (the maximum, and the default)

PostgreSQL accepts almost any reasonable `TIME` format including SQL-compatible, ISO 8601, and so on.

## Example 

It is pretty simple to create and use `TIME` data type, just use it like `int` and specify the values accordingly.

## TIME WITH TIME ZONE type 

Besides the `TIME` data type, PostgreSQL provides the `TIME WITH TIME ZONE` data type that allows you to store and manipulate the time of day with time zone.

Here's how to declare such a column:

```PostgreSQL
column TIME WITH TIME ZONE
```

The storage size of this type is 12 bytes, allowing you to store a time value with the time zone that ranges from `00:00:00+1459` to `24:00:00-1459`.

The following are some examples of the `TIME WITH TIME ZONE` type:

```
04:05:06 PST04:05:06.789-8
```

When dealing with time zone, it is recommended to use [`TIMESTAMP`](https://neon.com/postgresql/tutorial/postgresql-timestamp) instead of the `TIME WITH TIME ZONE` type.

## Handling PostgreSQL TIME values 

### 1) Getting the current time 

To get the current time with the time zone, you use the `CURRENT_TIME` function:

```PostgreSQL
SELECT CURRENT_TIME;
```

```
timetz
--------------------
 00:51:02.746572-08
(1 row)
```

To obtain the current time with a specific precision, you use the `CURRENT_TIME(precision)` function:

```PostgreSQL
SELECT CURRENT_TIME(5);
```

>Without specifying the precision, the `CURRENT_TIME` function returns a time value with the full available precision.

To get the local time, you use the `LOCALTIME` function:

```PostgreSQL
SELECT LOCALTIME;
```

Similarly, you can use `precision` here.

### 2) Converting time to a different time zone 

To convert time to a different time zone, you use the following form:

```PostgreSQL
[TIME with time zone] AT TIME ZONE time_zone
```

For example, to convert the local time to the time at the time zone UTC-7, you use the following statement:

```PostgreSQL
SELECT LOCALTIME AT TIME ZONE 'UTC-7';
```

### 3) Extracting hours, minutes, and seconds from a time value 

To extract hours, minutes, and seconds from a time value, you use the `EXTRACT` function as follows:

```PostgreSQL
EXTRACT(field FROM time_value);
```

The field can be the hour, minute, second, or milliseconds. For example:

```PostgreSQL
SELECT
    LOCALTIME,
    EXTRACT (HOUR FROM LOCALTIME) as hour,
    EXTRACT (MINUTE FROM LOCALTIME) as minute,
    EXTRACT (SECOND FROM LOCALTIME) as second,
    EXTRACT (milliseconds FROM LOCALTIME) as milliseconds;
```

### 4) Arithmetic operations on TIME 

You can apply arithmetic operator such as `+`, `-` or `*` on time values and between time and interval values.

The following returns an [interval](Interval%20Data%20Type.md) between two times:

```PostgreSQL
SELECT time '10:00' - time '02:00' AS result;
```

```
result
----------
 08:00:00
(1 row)
```

The following statement adds 2 hours to the local time:

```PostgreSQL
SELECT LOCALTIME + interval '2 hours' AS result;
```

```
result
-----------------
 03:16:18.020418
(1 row)
```

## Summary 

- Use the PostgreSQL `TIME` data type to store time data.