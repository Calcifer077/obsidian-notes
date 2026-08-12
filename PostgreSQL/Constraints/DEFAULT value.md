---
title: PostgreSQL DEFAULT value
source: https://neon.com/postgresql/tutorial/default-value
created: 2026-08-12
tags:
  - database
  - postgresql
---
When creating a table, you can define a default value for a column in the table using the `DEFAULT` constraint.

```PostgreSQL
CREATE TABLE table_name(
    column1 type,
    column2 type DEFAULT default_value,
    column3 type,
    ...
);
```

In this syntax, `column2` will receive `default_value` when you insert a new row into `table_name` without specifying value for that column.

If you don't specify `DEFAULT` constraint, its default value is `NULL`.

The default value can be a literal value such as a number, a string, a JSON object etc. It can also be an expression that will be evaluated at runtime (during insertion).

When inserting a new row into a table, you can ignore the column that has a default value. PostgreSQL will use the default value for insertion.

```PostgreSQL
INSERT INTO table_name(column1, column3)
VALUES(value1, value2);
```

If you specify the column with a default constraint in the `INSERT` statement and want to use the default value for insertion, you can use `DEFAULT` keyword.

```PostgreSQL
INSERT INTO table_name(column1, column2, column3)
VALUES(value1,DEFAULT,value2);
```

## `DEFAULT` value for a column of an existing table 

If you want to specify a default value for a column of an existing table, you can use `ALTER TABLE` statement.

```PostgreSQL
ALTER TABLE table_name
ALTER COLUMN column2
SET DEFAULT default_value;
```

## Removing the `DEFAULT` value from a column 

Use `ALTER` statement with `DROP` keyword:

```PostgreSQL
ALTER TABLE table_name
ALTER COLUMN column2
DROP DEFAULT;
```

## Summary 

- Use the `DEFAULT` constraint to define a default value for a table column.
- Use the `DEFAULT` keyword to explicitly use the default value specified in the `DEFAULT` constraint in the `INSERT` statement.