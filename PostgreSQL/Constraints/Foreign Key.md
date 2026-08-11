---
title: PostgreSQL Foreign Key
source: https://neon.com/postgresql/tutorial/foreign-key
created: 2026-08-11
tags:
  - database
  - postgresql
---
In PostgreSQL, a foreign key is a column or a group of columns in a table that uniquely identifies a row in **another table**. 

A foreign key establishes a link between the data in two tables by referencing the primary key or a unique constraint of the referenced table.

The table containing a foreign key is referred to as the referencing table or child table. Conversely, the table referenced by a foreign key is known as the referenced table or parent table.

The main purpose of foreign keys is to maintain referential integrity in a relational database, ensuring that relationships between the parent and child tables are valid.

For example, a foreign key prevents the insertion of values that do not have corresponding values in the referenced table.

Additionally, a foreign key maintains consistency by automatically updating or deleting related rows in the child table when changes occur in the parent table.

A table can have multiple foreign keys depending on its relationships with other tables.

## Syntax 

The following illustrates a foreign key constraint syntax:

```PostgreSQL
[CONSTRAINT fk_name]
   FOREIGN KEY(fk_columns)
   REFERENCES parent_table(parent_key_columns)
   [ON DELETE delete_action]
   [ON UPDATE update_action]
```

In this syntax:

- First, specify the name for the foreign key constraint after the `CONSTRAINT` keyword. The `CONSTRAINT` clause is optional. If you omit it, PostgreSQL will assign an auto-generated name.
- Second, specify one or more foreign key columns in parentheses after the `FOREIGN KEY` keywords.
- Third, specify the parent table and parent key columns referenced by the foreign key columns in the `REFERENCES` clause.
- Finally, specify the desired delete and update actions in the `ON DELETE` and `ON UPDATE` clauses.

The delete and update actions determine the behaviors when the primary key in the parent table is deleted and updated.

PostgreSQL supports the following actions:
- Set NULL
- Set DEFAULT 
- RESTRICT 
- NO ACTION 
- CASCADE

## Examples 

The following statement creates the `customers` and `contacts` tables:

```PostgreSQL
CREATE TABLE customers(
   customer_id INT GENERATED ALWAYS AS IDENTITY,
   customer_name VARCHAR(255) NOT NULL,
   PRIMARY KEY(customer_id)
);

CREATE TABLE contacts(
   contact_id INT GENERATED ALWAYS AS IDENTITY,
   customer_id INT,
   contact_name VARCHAR(255) NOT NULL,
   phone VARCHAR(15),
   email VARCHAR(100),
   PRIMARY KEY(contact_id),
   CONSTRAINT fk_customer
      FOREIGN KEY(customer_id)
        REFERENCES customers(customer_id)
);
```

In these tables, `customer_id` is primary key of `customers` and foreign key in `contacts`. `contact_id` is the primary key in `contacts`. `customer_id` of `contacts` references `customers.customer_id`.

Because the foreign key constraint does not have the `ON DELETE` and `ON UPDATE` action, they default to `NO ACTION`.

### NO ACTION 

The following inserts data into the `customers` and `contacts` tables:

```PostgreSQL
INSERT INTO customers(customer_name)
VALUES('BlueBird Inc'),
      ('Dolphin LLC');

INSERT INTO contacts(customer_id, contact_name, phone, email)
VALUES(1,'John Doe','(408)-111-1234','john.doe@example.com'),
      (1,'Jane Doe','(408)-111-1235','jane.doe@example.com'),
      (2,'David Wright','(408)-222-1234','david.wright@example.com');
```

The following statement deletes the `customer_id` 1 from the `customes` table:

```PostgreSQL
DELETE FROM customers
WHERE customer_id = 1;
```

Because of the `ON DELETE NO ACTION`, PostgreSQL issues a constraint violation because the referencing rows of the customer id 1 still exist in the `contacts` table:

```
ERROR:  update or delete on table "customers" violates foreign key constraint "fk_customer" on table "contacts"
DETAIL:  Key (customer_id)=(1) is still referenced from table "contacts".
SQL state: 23503
```

The difference between `RESTRICT` and `NO ACTION`  is that `NO ACTION` allows will check the constraint **after the operation** has been conceptually performed. If any child rows (referencing rows) still exist that would violate the foreign key, an error is raised and the operation is prevented. This is known as **deferred**. `RESTRICT` simply blocks it early on.

Here's a classic example of how **deferred** is useful:

1. You want to delete a row from the parent table.
2. Instead of deleting it, you update its primary key to a new value.
3. You then update all related child rows to point to this new key value.

### SET NULL 

The `SET NULL` automatically sets `NULL` to the foreign key columns in the referencing rows of the child table when the references rows in the parent table are deleted. 

Here's how it would look in syntax:

```PostgreSQL
DROP TABLE IF EXISTS contacts;
DROP TABLE IF EXISTS customers;

CREATE TABLE customers(
   customer_id INT GENERATED ALWAYS AS IDENTITY,
   customer_name VARCHAR(255) NOT NULL,
   PRIMARY KEY(customer_id)
);

CREATE TABLE contacts(
   contact_id INT GENERATED ALWAYS AS IDENTITY,
   customer_id INT,
   contact_name VARCHAR(255) NOT NULL,
   phone VARCHAR(15),
   email VARCHAR(100),
   PRIMARY KEY(contact_id),
   CONSTRAINT fk_customer
      FOREIGN KEY(customer_id)
	  REFERENCES customers(customer_id)
	  ON DELETE SET NULL
);
```

### CASCADE 

The `ON DELETE CASCADE` automatically deletes all the referencing rows in the child table when the referenced rows in the parent table are deleted. 

Syntax:

```PostgreSQL
CONSTRAINT fk_customer
      FOREIGN KEY(customer_id)
	  REFERENCES customers(customer_id)
	  ON DELETE CASCADE
```

### SET DEFAULT 

The `ON DELETE SET DEFAULT` sets the default value to the foreign key column of the referencing rows in the child table when the referenced rows from the parent table are deleted.

```PostgreSQL
-- 1. Create the parent table
CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY,
    DeptName VARCHAR(50) NOT NULL
);

-- 2. Create the child table with the SET DEFAULT constraint
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    EmpName VARCHAR(50) NOT NULL,
    DepartmentID INT DEFAULT 1, -- Define the column default value

    CONSTRAINT FK_Employee_Department 
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
    ON DELETE SET DEFAULT -- Fall back to default when parent row is deleted
    ON UPDATE SET DEFAULT -- Fall back to default when parent ID changes
);
```

## Add a foreign key constraint to an existing table 

To add a foreign key constraint to an existing table, you use the following form of the `ALTER TABLE` statement:

```PostgreSQL
ALTER TABLE child_table
ADD CONSTRAINT constraint_name
FOREIGN KEY (fk_columns)
REFERENCES parent_table (parent_key_columns);
```

When adding a foreign key constraint with `ON DELETE CASCADE` option to an existing table, you first need to drop existing foreign key constraint before adding any new one.

```PostgreSQL
ALTER TABLE child_table
DROP CONSTRAINT constraint_fkey;

ALTER TABLE child_table
ADD CONSTRAINT constraint_fk
FOREIGN KEY (fk_columns)
REFERENCES parent_table(parent_key_columns)
ON DELETE CASCADE;
```

## Summary 

- Use foreign keys to ensure the referential integrity and consistency of data between two tables.
- Use the `FOREIGN KEY` constraint to define a foreign key constraint when creating a table.
- Use the `ALTER TABLE ... ADD CONSTRAINT ... FOREIGN KEY` to add a foreign key constraint to an existing table.