---
title: PostgreSQL DOUBLE PRECISION Data Type
source: https://neon.com/postgresql/tutorial/double-precision-type
created: 2026-10-14
tags:
  - postgresql
---
`DOUBLE PRECISION` is an inexact, variable-precision numeric type. 

Inexact means that PostgreSQL cannot exactly convert some values into an internal format and can only store them as approximations. This inexact is the same one you get when you do `0.1 + 0.2` and it doesn't equate to `0.3` in JS.

```PostgreSQL
SELECT 0.1::double precision + 0.2::double precision = 0.3::double precision;
-- false

SELECT (0.1::double precision + 0.2::double precision)::numeric(20,17);
-- 0.30000000000000004
```

It is because decimals can't be represented exactly in binary.

If your application requires exact storage and calculation, it's recommended to use the [`numeric`](Numeric%20Type.md) type instead.

The following shows how to define a column with the `DOUBLE PRECISION` type:

```PostgreSQL
column_name double precision
```

Alternatively, you can use the `float8` or `float` data type which is the same as `double precision`.

A column of `DOUBLE PRECISION` type can store values that have a range between `1E-307` to `1E+308` with a precision of at least 15 digits.

If you store a value that is out of range, PostgreSQL will be unable to store it and raise an error.

If you store numbers with very high precision, PostgreSQL may round them to fit within the limitation of double precision.

If you store very small numbers close to zero, PostgreSQL may raise an underflow error due to the limitations of double precision data type, which may be unable to accurately represent such small values distinct from zero.

## Summary 

- `DOUBLE PRECISION` data type represents the inexact numbers.
- `DOUBLE PRECISION`, `FLOAT8`, or `FLOAT` are the same.
- Use `DOUBLE PRECISION` type to store inexact numbers and `NUMERIC` type to store exact numbers. 