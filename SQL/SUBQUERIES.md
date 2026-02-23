A subquery in SQL is a query nested inside another SQL query.

The query inside '()' is a subquery. What it does is the subquery will return some rows and the outer query will try to match those rows. Subquery can only select one row.
**`select *`** 
**`from employee_demographics`**
**`where employee_id in`** 
**`(`**
	**`select employee_id`**
	**`from employee_salary`**
	**`where dept_id = 1`**
**`);`**

Subquery inside select statement.
**`select first_name, salary,`**
**`(`**
	**`select avg(salary) from employee_salary`**
**`) as avg_salary`**
**`from employee_salary;`**

Subquery inside from. Below we are applying aggregate functions on some result provided by a aggregate function.
**`select max(max_age)`**
**`from`**
**`(`**
	**`select`** 
	**`gender as gen,`** 
	**`avg(age) as avg_age,`** 
	**`max(age) as max_age,`** 
	**`min(age) as min_age,`** 
	**`count(age) as count_age`**
	**`from employee_demographics`**
	**`group by gender`**
**`) as agg_table;`**