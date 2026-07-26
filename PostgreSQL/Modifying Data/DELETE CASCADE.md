---
title: PostgreSQL DELETE CASCADE
source: https://neon.com/postgresql/tutorial/delete-cascade
created: 2026-07-24
tags:
  - database
  - postgresql
---
The `DELETE CASCADE` is a referential action that allows us to automatically **delete** related rows in child tables when a parent row is deleted from the parent table.

This feature helps us maintain referential integrity in the database by ensuring that dependent rows are removed when their corresponding rows are deleted.

To enable `DELETE CASCADE` action, we need to have two related tables `parent_table` and `child_table`:

```PostgreSQL
CREATE TABLE parent_table(
    id SERIAL PRIMARY KEY,
    ...
);

CREATE TABLE child_table(
    id SERIAL PRIMARY KEY,
    parent_id INT,
    FOREIGN_KEY(parent_id)
       REFERENCES parent_table(id)
       ON DELETE CASCADE
);
```

In the child table, the `parent_id` is a foreign key that references the `id` column of the `parent_table`. The `ON DELETE CASCADE` is the action on the foreign key that will automatically delete the rows from the `child_table` whenever corresponding rows from the `parent_table` are deleted.

## Examples

Let's create sample tables named `departments` and `employees`:

```PostgreSQL
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department_id INT NOT NULL,
    FOREIGN KEY(department_id)
       REFERENCES departments(id)
       ON DELETE CASCADE
);

INSERT INTO departments (name)
VALUES
    ('Engineering'),
    ('Sales')
RETURNING *;

INSERT INTO employees (name, department_id)
VALUES
    ('John Doe', 1),
    ('Jane Smith', 1),
    ('Michael Johnson', 2)
RETURNING *;
```

Output:

```
id |    name
----+-------------
  1 | Engineering
  2 | Sales
(2 rows)

 id |      name       | department_id
----+-----------------+---------------
  1 | John Doe        |             1
  2 | Jane Smith      |             1
  3 | Michael Johnson |             2
(3 rows)
```

In the `employees` table, the `department_id` is a foreign key that references the id column of the `departments` table. The foreign key has the `ON DELETE CASCADE` clause that specifies the referential action to take when a row in the `departments` table is deleted.

Let's try to delete a department.

```PostgreSQL
DELETE FROM departments
WHERE id = 1;
```

Once we execute above statement, it deletes all employees belonging to the department with `department_id` = 1 due to the `DELETE CASCADE` action defined on the foreign key constraint.

We can verify that the operation succeeded or not:

```PostgreSQL
SELECT * FROM employees;
```

Output:

```
id |      name       | department_id
----+-----------------+---------------
  3 | Michael Johnson |             2
(1 row)
```

## Summary

- Use PostgreSQL `DELETE CASCADE` action to automatically delete related rows in child tables when a parent row is deleted.