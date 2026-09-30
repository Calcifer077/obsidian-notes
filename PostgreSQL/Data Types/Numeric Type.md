---
title: PostgreSQL Numeric Type
source: https://neon.com/postgresql/tutorial/numeric
created: 2026-09-29
tags:
  - postgresql
---
The `NUMERIC` type can store numbers with lot of digits. 

Here's the syntax for declaring a column with the `NUMERIC` type:

```PostgreSQL
column_name NUMERIC(precision, scale)
```

In this syntax,

- The `precision` is the total number of digits.
- The `scale` is the number of digits in the fraction part.

The `NUMERIC` type can hold a value of up to `131,072` digits before the decimal point `16,383` digits after the decimal point.

The scale of the `NUMERIC` type can be zero, positive, or negative. Below are simple examples for different values of scale.

|Type|Scale|Input|Stored|Notes|
|---|---|---|---|---|
|`numeric(5,2)`|Positive|`123.456`|`123.46`|Rounds to 2 decimal places|
|`numeric(5,2)`|Positive|`0.5`|`0.50`|Padded with zeros|
|`numeric(5,2)`|Positive|`1234.5`|ERROR|Overflow, max is `999.99`|
|`numeric(5,0)`|Zero|`123.456`|`123`|Rounds to a whole number|
|`numeric(5,0)`|Zero|`12345.6`|`12346`|Rounds up|
|`numeric(5,0)`|Zero|`123456`|ERROR|Overflow, max is `99999`|
|`numeric(5,-2)`|Negative|`12345`|`12300`|Rounds to nearest hundred|
|`numeric(5,-2)`|Negative|`12350`|`12400`|Half rounds away from zero|
|`numeric(3,-3)`|Negative|`1234567`|`1235000`|Rounds to nearest thousand|
|`numeric(3,-3)`|Negative|`1000000000`|ERROR|Overflow, max is `999000`|

>Error will be of type 'numeric field overflow'.

## NUMERIC, DECIMAL, and DEC types

All three `NUMERIC`, `DECIMAL` and `DEC` are same. They just differ in name. 

If precision is not required, you should not use the `NUMERIC` (not even `NUMERIC(precision, 0)`) type because calculations on `NUMERIC` values are typically slower than integers, float and double precisions.

### Special values 

Besides the ordinal numeric values, the `numeric` type has several special values:

- `Infinity`
- `-Infinity`
- `NaN`

These values represent “infinity”, “negative infinity”, and “not-a-number”, respectively.

Typically (in JS), `NaN` are not equal to anything including itself. But this is not true in PostgreSQL, two `NaN` are equal to each other. Also note that `NaN` is the biggest number of all, even bigger than `Infinity`.

## Summary 

- Use the PostgreSQL`NUMERIC` data type to store numbers that require exactness.