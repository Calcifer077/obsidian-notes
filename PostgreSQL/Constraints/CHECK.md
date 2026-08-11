---
title: PostgreSQL CHECK constraints
source: https://neon.com/postgresql/tutorial/check-constraint
created: 2026-08-11
tags:
  - database
  - postgresql
---
A `CHECK` constraint ensures that values in a column or a group of columns meet a specific condition.

A check constraint allows you to enforce data integrity rules at the database level. A check constraint uses a boolean expression to evaluate the values, ensuring that only valid data is inserted or updated in a table. 

## Creating CHECK constraints 

Normally, you create check constraint while creating a table.

```PostgreSQL
CREATE TABLE table_name(
   column1 datatype,
   ...,
   CONSTRAINT constraint_name CHECK(condition)
);
```

In this syntax:

- First, specify the constraint name after the `CONSTRAINT` keyword. This is optional. If you omit it, PostgreSQL will automatically generate a name for the `CHECK` constraint.
- Second, define a condition that must be satisfied for the constraint to be valid.

If the `CHECK` constraint involves only one column, you can define it as a column constraint like this:

```PostgreSQL
CREATE TABLE table_name(
   column1 datatype,
   column1 datatype CHECK(condition),
   ...,
);
```

By default, PostgreSQL assigns a name to a `CHECK` constraint using the following format:

```
{table}_{column}_check
```

To add a `CHECK` constraint to an existing table, you can use `ALTER` statement:

```PostgreSQL
ALTER TABLE table_name
ADD CONSTRAINT constraint_name CHECK (condition);
```

To drop a `CHECK` constraint:

```PostgreSQL
ALTER TABLE table_name
DROP CONSTRAINT constraint_name;
```

## Examples 

### 1) For a new table 

Create a new table called `employees` with some `CHECK` constraints:

```PostgreSQL
CREATE TABLE employees (
  id SERIAL PRIMARY KEY,
  first_name VARCHAR (50) NOT NULL,
  last_name VARCHAR (50) NOT NULL,
  birth_date DATE NOT NULL,
  joined_date DATE NOT NULL,
  salary numeric CHECK(salary > 0)
);
```

In this statement, `employees` table has one `CHECK` constraint that enforces the values in `salary` column to be greater than zero. If you attempt to insert any negative values, PostgreSQL will issue an error.

```PostgreSQL
INSERT INTO employees (first_name, last_name, birth_date, joined_date, salary)
VALUES ('John', 'Doe', '1972-01-01', '2015-07-01', -100000);
```

Error: 

```
ERROR:  new row for relation "employees" violates check constraint "employees_salary_check"
DETAIL:  Failing row contains (1, John, Doe, 1972-01-01, 2015-07-01, -100000).
```

### 2) Adding for existing tables 

We can use `ALTER` statement for this purpose:

```PostgreSQL
ALTER TABLE employees
ADD CONSTRAINT joined_date_check
CHECK ( joined_date >  birth_date );
```

The `CHECK` constraint ensures that the `joined_date` is later then the `birthdate`.

```PostgreSQL
INSERT INTO employees (first_name, last_name, birth_date, joined_date, salary)
VALUES ('John', 'Doe', '1990-01-01', '1989-01-01', 100000);
```

Error:

```
ERROR:  new row for relation "employees" violates check constraint "joined_date_check"
DETAIL:  Failing row contains (2, John, Doe, 1990-01-01, 1989-01-01, 100000).
```

### 3) Using functions 

The following example adds a `CHECK` constraint that makes sure that the first name has at least 3 characters:

```PostgreSQL
ALTER TABLE employees
ADD CONSTRAINT first_name_check
CHECK ( LENGTH(TRIM(first_name)) >= 3);
```

### 4) Removing a CHECK constraint 

The following statement removes the `CHECK` constraint `joined_date_check` (created in second example) from the `employees` table.

```PostgreSQL
ALTER TABLE employees
DROP CONSTRAINT joined_date_check;
```

## Summary 

- Use PostgreSQL `CHECK` constraint to check the values of columns based on a boolean expression.