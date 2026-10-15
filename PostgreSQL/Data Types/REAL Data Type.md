---
title: PostgreSQL REAL Data Type
source: https://neon.com/postgresql/tutorial/real-data-type
created: 2026-10-15
tags:
  - postgresql
---
The `REAL` data type allows you to store single-precision floating-point numbers in the database.

A value of the real type takes 4 bytes of storage space. It's valid range is about 1E-37 to 1E+37.

`REAL` can reliably only store 6 significant bits. After which it can lead to unexpected results (like rounded off numbers).

You can declare a `REAL` column by following syntax:

```PostgreSQL
column_name REAL;
```

## Example 

```PostgreSQL
CREATE TABLE weathers(
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    location VARCHAR(255) NOT NULL,
    wind_speed_mps REAL NOT NULL,
    temperature_celsius REAL NOT NULL,
    recorded_at TIMESTAMP NOT NULL
);

INSERT INTO weathers (location, wind_speed_mps, temperature_celsius, recorded_at)
VALUES
    ('New York', 5.2, 15.3, '2024-04-19 09:00:00'),
    ('New York', 4.8, 14.9, '2024-04-19 10:00:00'),
    ('New York', 6.0, 16.5, '2024-04-19 11:00:00'),
    ('New York', 5.5, 15.8, '2024-04-19 12:00:00'),
    ('New York', 4.3, 14.2, '2024-04-19 13:00:00'),
    ('New York', 5.9, 16.1, '2024-04-19 14:00:00'),
    ('New York', 6.8, 17.3, '2024-04-19 15:00:00'),
    ('New York', 5.1, 15.6, '2024-04-19 16:00:00'),
    ('New York', 4.7, 14.8, '2024-04-19 17:00:00'),
    ('New York', 5.3, 15.9, '2024-04-19 18:00:00');
```

If you calculate the average of wind speed and temperature you would get something like this:

```PostgreSQL
SELECT
  AVG(wind_speed_mps)       wind_speed,
  AVG(temperature_celsius) temperature_celsius
FROM
  weathers
WHERE
  location = 'New York'
  AND DATE(recorded_at) = '2024-04-19';
```

Output:

```
wind_speed     | temperature_celsius
-------------------+---------------------
 5.360000038146973 |  15.639999961853027
(1 row)
```

## Summary 

- Use the PostgreSQL `REAL` data type to store single-precision floating-point numbers in the database.