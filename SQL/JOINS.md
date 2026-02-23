A `JOIN` clause is used to combine columns from two or more tables, based on a related column between them.

INNER JOIN

The `INNER JOIN` keyword selects records that have matching values in both tables.

**`select *`** 
**`from employee_demographics`**
**`inner join employee_salary`**
**`on employee_id = employee_id;`**

This will give an error saying **`employee_id`** is ambiguous which mean that it doesn't know which tables **`employee_id`**. You will get the same error even when you are trying to access columns in the **`select`** statement, which you can avoid by using aliases. Ambiguity will only occur for those columns which are similar for both the tables, if there are different columns name than there will be no problem.

Solution to above problem:
**`select *`** 
**`from employee_demographics as demo`**
**`inner join employee_salary as sal`**
**`on demo.employee_id = sal.employee_id;`**

**`select demo.first_name, age, occupation`**
**`from employee_demographics as demo`**
**`inner join employee_salary as sal`**
**`on demo.employee_id = sal.employee_id;`**

It basically fetches all those rows which satisfies **`demo.employee_id = sal.employee_id`**, if there is a row which doesn't satisfy it, it won't be included in the result.

OUTER JOIN
is of two types, Left outer join (left join) and right outer join (right join).

The `LEFT JOIN` keyword returns all records from the left table (table1), and the matching records from the right table (table2). The result is 0 records from the right side, if there is no match.

The `RIGHT JOIN` keyword returns all records from the right table (table2), and the matching records from the left table (table1). The result is 0 records from the left side, if there is no match.

**`select *`**
**`from employee_demographics as demo`**
**`left join employee_salary as sal`**
**`on demo.employee_id = sal.employee_id;`**

**`select *`**
**`from employee_demographics as demo`**
**`right join employee_salary as sal`**
**`on demo.employee_id = sal.employee_id;`**


SELF JOIN
A self join is a regular join, but the table is joined with itself.
**`select *`**
**`from employee_salary as emp1`**
**`join employee_salary as emp2`**
**`on emp1.employee_id + 1 = emp2.employee_id;`**

Joining Multiple tables together
**`select *`**
**`from employee_demographics as dem`**
**`inner join employee_salary as sal`**
	**`on dem.employee_id = sal.employee_id`**
**`inner join parks_departments as pd`**
	**`on sal.dept_id = pd.department_id;`**