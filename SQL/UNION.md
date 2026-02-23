Combines **rows** from two queries into a single result set. You can combine rows from multiple tables or a single table.

**`select first_name, last_name`**
**`from employee_demographics`**
**`union`** 
**`select first_name, last_name`**
**`from employee_salary;`**

If you try to union on different datatypes it will give you error saying that it can't convert.  For it to work, the number of columns that we are querying should be same for all the queries. For example in the above query we are getting first_name and last_name from both. If there was one less or more it will give error.

By default it only gives unique values. If you want all the values whether they are unique or not.
**`select first_name, last_name`**
**`from employee_demographics`**
**`union all`**
**`select first_name, last_name`**
**`from employee_salary;`**

Example of using Union:
**`select first_name, last_name, 'Old Man' as [Label]`**
**`from employee_demographics`**
**`where age > 40 and gender = 'Male'`**
**`union`**
**`select first_name, last_name, 'Old Lady' as [Label]`**
**`from employee_demographics`**
**`where age > 40 and gender = 'Female'`**
**`union`**
**`select first_name, last_name, 'Highly Paid Employee' as [Label]`**
**`from employee_salary`**
**`where salary > 70000`**
**`;`**

