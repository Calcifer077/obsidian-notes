---
title: PostgreSQL DATE Data Type
source: https://neon.com/postgresql/tutorial/date
created: 2026-10-17
tags:
  - postgresql
---
PostgreSQL offers the `DATE` data type that allows you to store date data.

PostgreSQL uses 4 bytes to store a date value. The lowest and highest values of the `DATE` data type are 4713 BC and 5874897 AD, respectively.

When storing a date value, PostgreSQL uses the  `yyyy-mm-dd` format such as 2000-12-31. It also uses the same format for [inserting data](https://neon.com/postgresql/tutorial/postgresql-insert) into a `DATE` column.

If you want to use the current date as the default value, you can use the `CURRENT_DATE` as the default value of the column using a `DEFAULT` constraint.

```PostgreSQL
CREATE TABLE documents (
  document_id SERIAL PRIMARY KEY,
  header_text VARCHAR (255) NOT NULL,
  posting_date DATE NOT NULL DEFAULT CURRENT_DATE
);
```

Now, if you don't give any value while inserting `posting_date`, default value will be used. 

## PostgreSQL DATE functions 

For demonstration, we will create a new table `employees`:

```PostgreSQL
CREATE TABLE employees (
  employee_id SERIAL PRIMARY KEY,
  first_name VARCHAR (255) NOT NULL,
  last_name VARCHAR (255) NOT NULL,
  birth_date DATE NOT NULL,
  hire_date DATE NOT NULL
);

INSERT INTO employees (first_name, last_name, birth_date, hire_date)
VALUES ('Shannon','Freeman','1980-01-01','2005-01-01'),
       ('Sheila','Wells','1978-02-05','2003-01-01'),
       ('Ethel','Webb','1975-01-01','2001-01-01')
RETURNING *;
```

Here, `birth_date`, `hire_date` are date columns.

> The result of some functions may differ based on your database server's date.

### 1) Get the current date 

To get the current date and time, you can use the built-in `NOW()` function:

```PostgreSQL
SELECT NOW();
```

Output:

```
now
-------------------------------
 2024-02-01 08:48:09.599933+07
(1 row)
```

To get the date part only (without the time part), you use the cast operator (::) to case a `TIMESTAMP` value to a `DATE` value:

```PostgreSQL
SELECT NOW():date;
```

Output:

```
now
------------
 2024-02-01
(1 row)
```

A quick way to get the current date is to use the `CURRENT_DATE` function:

```PostgreSQL
SELECT CURRENT_DATE;
```

Output:

```
current_date
--------------
 2024-02-01
(1 row)
```

The result is in the format `yyyy-mm-dd`. However, you can use a different format by formatting the date value using the `TO_CHAR()` function. 

### 2) Output a PostgreSQL date value in a specific format 

To output a date value in a specific format, you use the `TO_CHAR()` function.

The `TO_CHAR()` function accepts two parameters. The first parameter is the value you want to format, and the second is the template that defines the output format.

```PostgreSQL
SELECT TO_CHAR(CURRENT_DATE, 'dd/mm/yyyy');
```

```
to_char
------------
 01/02/2024
(1 row)
```

To display a date in a format like `Feb 01, 2024`, you use the following statement:

```PostgreSQL
SELECT TO_CHAR(CURRENT_DATE, 'Mon dd, yyyy');
```

```
to_char
--------------
 Feb 01, 2024
(1 row)
```

### 3) Get the interval between two dates 

To get the interval between two dates, you use the minus (`-`) operator.

```PostgreSQL
SELECT
  first_name,
  last_name,
  now() - hire_date as diff
FROM
  employees;
```

Output:

```
first_name | last_name |           diff
------------+-----------+---------------------------
 Shannon    | Freeman   | 6970 days 08:51:20.824847
 Sheila     | Wells     | 7701 days 08:51:20.824847
 Ethel      | Webb      | 8431 days 08:51:20.824847
(3 rows)
```

### 4) Calculate ages in years, months, and days 

To calculate age at the current date in years, months, and days, you use the `AGE()` function.

```PostgreSQL
SELECT
	employee_id,
	first_name,
	last_name,
	AGE(birth_date)
FROM
	employees;
```

Output:

```
employee_id | first_name | last_name |           age
-------------+------------+-----------+--------------------------
           1 | Shannon    | Freeman   | 44 years 1 mon
           2 | Sheila     | Wells     | 45 years 11 mons 24 days
           3 | Ethel      | Webb      | 49 years 1 mon
(3 rows)
```

If you pass a date value to the `AGE()` function, it will subtract the date value from the current date.

If you pass two arguments to the `AGE()` function, it will subtract the second argument from the first argument.

### 5) Extract year, quarter, month, week, and day from a date value 

To get the year, quarter, month, week, and day from a date value, you use the `EXTRACT()` function.

```PostgreSQL
SELECT
	employee_id,
	first_name,
	last_name,
	EXTRACT (YEAR FROM birth_date) AS YEAR,
	EXTRACT (MONTH FROM birth_date) AS MONTH,
	EXTRACT (DAY FROM birth_date) AS DAY
FROM
	employees;
```

Output:

```
employee_id | first_name | last_name | year | month | day
-------------+------------+-----------+------+-------+-----
           1 | Shannon    | Freeman   | 1980 |     1 |   1
           2 | Sheila     | Wells     | 1978 |     2 |   5
           3 | Ethel      | Webb      | 1975 |     1 |   1
(3 rows)
```

