In PostgreSQL, a [boolean](https://neon.com/postgresql/tutorial/postgresql-boolean) value can have one of three values: `true`, `false`, and `null`.

PostgreSQL uses `true`, `'t'`, `'true'`, `'y'`, `'yes'`, `'1'` to represent `true` and `false`, `'f'`, `'false'`, `'n'`, `'no'`, and `'0'` to represent `false`.

A boolean expression is an expression that evaluates to a boolean value. For example, the expression `1=1` is a boolean expression that evaluates to `true`:
```PostgreSQL
SELECT 1 = 1 AS result;
```

Output:
```
result
--------
 t
(1 row)
```

The letter `t` in the output indicates the value of `true`. 

The `AND` operator is a logical operator that combines two boolean expressions.

Basic syntax:
```PostgreSQL
expression1 AND expression2
```

In this syntax, `expression1` and `expression2` are boolean expressions that evaluate to `true`, `false`, or `null`.

The `AND` operator returns `true` only if both expressions are `true`. It returns `false` if one of the expressions is `false`. **Otherwise, it returns `null`.** If any expression is `null`, it doesn't matter whether other expression is `true`, `false,` or `null` result will be `null`.

## Examples

### 1. Basics examples
The following example uses the `AND` operator to combine two true values, which returns true:
```PostgreSQL
SELECT true AND true AS result;
```

Output:
```
result
--------
 t
(1 row)
```

The following statement uses the `AND` operator to combine true with false, which returns false:
```PostgreSQL
SELECT true AND false AS result;
```

The following example uses the `AND` operator to combine true with null, which returns null:
```PostgreSQL
SELECT true AND null AS result;
```

The following example uses the `AND` operator to combine false with false, which returns false:
```PostgreSQL
SELECT false AND false AS result;
```

The following example uses the `AND` operator to combine false with null, which returns false:
```PostgreSQL
SELECT false AND null AS result;
```

The following example uses the `AND` operator to combine null with null, which returns null:
```PostgreSQL
SELECT null AND null AS result;
```

### 2. Using in WHERE clause
We'll use the `film` table from the [Quick Start - Settings things up](../Quick%20Start%20-%20Setting%20things%20up/Quick%20Start%20-%20Settings%20things%20up.md) section.

![](../../assets/film_table_dvdrental_and_operator_neon_postgres_sql_tutorial.png)

The following example uses the `AND` operator in the `WHERE` clause to find the films that have a length greater than 180 and a rental rate less than 1:
```PostgreSQL
SELECT
  title,
  length,
  rental_rate
FROM
  film
WHERE
  length > 180
  AND rental_rate < 1;
```

Output:
```
title        | length | rental_rate
--------------------+--------+-------------
 Catch Amistad      |    183 |        0.99
 Haunting Pianist   |    181 |        0.99
 Intrigue Worst     |    181 |        0.99
 Love Suicides      |    181 |        0.99
 Runaway Tenenbaums |    181 |        0.99
 Smoochy Control    |    184 |        0.99
 Sorority Queen     |    184 |        0.99
 Theory Mermaid     |    184 |        0.99
 Wild Apollo        |    181 |        0.99
 Young Language     |    183 |        0.99
(10 rows)
```

## Summary

- Use the `AND` operator to combine multiple boolean expressions.

## Sources

[Neon - and operator](https://neon.com/postgresql/tutorial/and)

## Tags
#database  #postgresql 
