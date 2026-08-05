---
title: PostgreSQL Change Column's data Type
source: https://neon.com/postgresql/tutorial/change-column-type
created: 2026-08-05
tags:
  - database
  - postgresql
---
To change the data type of a column, you can use the [`ALTER TABLE`](ALTER%20TABLE.md) statement as follows:

```PostgreSQL
ALTER TABLE table_name
ALTER COLUMN column_name
[SET DATA] TYPE new_data_type;
```

In this syntax:

- First, specify the name of the table to which the column you want to change after the `ALTER TABLE` keywords.
- Second, provide the name of the column that you want to change the data type after the `ALTER COLUMN` clause.
- Third, supply the new data type for the column after the `TYPE` keyword. The `SET DATA TYPE` and `TYPE` are equivalent.

To change the data types of multiple columns in a single statement, you use multiple `ALTER COLUMN` clauses like this:

```PostgreSQL
ALTER TABLE table_name
ALTER COLUMN column_name1 [SET DATA] TYPE new_data_type,
ALTER COLUMN column_name2 [SET DATA] TYPE new_data_type,
...;
```

PostgreSQL allows you to convert the value of a column to the new ones while changing its data type by adding a `USING` clause as follows:

```PostgreSQL
ALTER TABLE table_name
ALTER COLUMN column_name TYPE new_data_type USING expression;
```

The `USING` clause specifies an expression that allows you to convert the old values to the new ones.

If you omit the `USING` clause, PostgreSQL will cast the values to the new ones implicitly. If the cast fails, PostgreSQL will issue an error and recommend you provide the `USING` clause with an expression for the data conversion.

The expression after the `USING` keyword can be as simple as `column_name::new_data_type` such as `price::numeric` or as complex as a custom function.

## Examples

Let's first setup sample data.

```PostgreSQL
CREATE TABLE assets (
    id serial PRIMARY KEY,
    name TEXT NOT NULL,
    asset_no VARCHAR NOT NULL,
    description TEXT,
    location TEXT,
    acquired_date DATE NOT NULL
);

INSERT INTO assets(name,asset_no,location,acquired_date)
VALUES('Server','10001','Server room','2017-01-01'),
      ('UPS','10002','Server room','2017-01-01')
RETURNING *;
```

### 1) Changing one column

The following example changes the data type of the `name` column to `VARCHAR`:

```PostgreSQL
ALTER TABLE assets
ALTER COLUMN name TYPE VARCHAR(255);
```

### 2) Changing a column from `VARCHAR` to `INT`

```PostgreSQL
ALTER TABLE assets
ALTER COLUMN asset_no TYPE INT;
```

PostgreSQL will issue an error that looks something like this:

```
ERROR:  column "asset_no" cannot be cast automatically to type integer
HINT:  You might need to specify "USING asset_no::integer".
```

To both change the type of a column and cast data from `VARCHAR` to `INT`, you can use the `USING` clause:

```PostgreSQL
ALTER TABLE assets
ALTER COLUMN asset_no TYPE INT
USING asset_no::integer;
```

## Summary 

- Use the `ALTER TABLE ... ALTER COLUMN` statement to change the data type of a column.