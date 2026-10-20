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

