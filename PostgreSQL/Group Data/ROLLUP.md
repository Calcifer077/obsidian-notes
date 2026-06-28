---
title: PostgreSQL ROLLUP
source: https://neon.com/postgresql/tutorial/rollup
created: 2026-06-28
description: In this tutorial, you will learn how to use the PostgreSQL ROLLUP to generate multiple grouping sets.
tags:
  - database
  - postgresql
---
The PostgreSQL `ROLLUP` is a subclause of the [`GROUP BY`](GROUP%20BY.md) clause that offers a shorthand for defining multiple [grouping sets](GROUPING%20SETS.md). A grouping set is a set of columns by which you group. 

Different from the [`CUBE`](CUBE.md) subclause, `ROLLUP` does not generate all possible grouping sets based on the specified columns. It just makes a subset of those.

The `ROLLUP` assumes a hierarchy among the input columns and generates all grouping sets that make sense considering the hierarchy. This is the reason why `ROLLUP` is often used to generate the subtotals and the grand total for reports.

For example, the `CUBE (c1,c2,c3)` makes all eight possible grouping sets:

```text
(c1, c2, c3)
(c1, c2)
(c2, c3)
(c1,c3)
(c1)
(c2)
(c3)
()
```

However, the `ROLLUP(c1,c2,c3)` generates only four grouping sets, assuming the hierarchy `c1 > c2 > c3` as follows:

```text
(c1, c2, c3)
(c1, c2)
(c1)
()
```

A common use of `ROLLUP` is to calculate the aggregations of data by year, month, and date, considering the hierarchy `year > month > date`

The following illustrates the syntax of the PostgreSQL `ROLLUP`:

```sql
SELECT
    c1,
    c2,
    c3,
    aggregate(c4)
FROM
    table_name
GROUP BY
    ROLLUP (c1, c2, c3);
```

It is also possible to do a partial roll up to reduce the number of subtotals generated.

```sql
SELECT
    c1,
    c2,
    c3,
    aggregate(c4)
FROM
    table_name
GROUP BY
    c1,
    ROLLUP (c2, c3);
```

## PostgreSQL ROLLUP examples

These examples use the `sales` table from [`GROUPING SETS`](GROUPING%20SETS.md) note.

The following query uses the `ROLLUP` clause to find the number of products sold by brand (subtotal) and by all brands and segments (total).

```sql
SELECT
    brand,
    segment,
    SUM (quantity)
FROM
    sales
GROUP BY
    ROLLUP (brand, segment)
ORDER BY
    brand,
    segment;
```

![](../../assets/Pasted%20image%2020260628091938.png)

As you can see clearly from the output, the third row shows the sales of the `ABC` brand, the sixth row displays sales of the `XYZ` brand. The last row shows the grand total for all brands and segments. In this example, the hierarchy is `brand > segment`.

If you change the order of brand and segment, the result will be different as follows:

```sql
SELECT
    segment,
    brand,
    SUM (quantity)
FROM
    sales
GROUP BY
    ROLLUP (segment, brand)
ORDER BY
    segment,
    brand;
```

![](../../assets/Pasted%20image%2020260628092012.png)

In this case, the hierarchy is the `segment > brand`.

The following statement performs a partial roll-up:

```sql
SELECT
    segment,
    brand,
    SUM (quantity)
FROM
    sales
GROUP BY
    segment,
    ROLLUP (brand)
ORDER BY
    segment,
    brand;
```

![](../../assets/Pasted%20image%2020260628092039.png)

See the following `rental` table from the [Quick Start - Settings things up](../Quick%20Start%20-%20Setting%20things%20up/Quick%20Start%20-%20Settings%20things%20up.md).

![](../../assets/Pasted%20image%2020260628092056.png)

The following statement finds the number of rentals per day, month, and year by using the `ROLLUP`:

```sql
SELECT
    EXTRACT (YEAR FROM rental_date) y,
    EXTRACT (MONTH FROM rental_date) M,
    EXTRACT (DAY FROM rental_date) d,
    COUNT (rental_id)
FROM
    rental
GROUP BY
    ROLLUP (
        EXTRACT (YEAR FROM rental_date),
        EXTRACT (MONTH FROM rental_date),
        EXTRACT (DAY FROM rental_date)
    );
```

In above code `EXTRACT` retrieves specific part of a date or timestamp. 

`ROLLUP` in above case creates the following grouping sets:

```text
(Year, Month, Day)   -- Daily totals
(Year, Month)        -- Monthly totals
(Year)               -- Yearly totals
()                   -- Grand total
```

![](../../assets/Pasted%20image%2020260628092122.png)

In this tutorial, you have learned how to use the PostgreSQL `ROLLUP` to generate multiple grouping sets.

## Summary

- Use `ROLLUP` when you don't want all the grouping sets that are created by `CUBE` but instead want some hierarchical sets.