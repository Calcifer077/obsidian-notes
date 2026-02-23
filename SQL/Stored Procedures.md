These are basically like functions which stores some queries which you can execute at a later time.

Creating a simple stored procedure. When you execute this stored procedure using `exec` it will give you the result of the query inside it. In this case being the `select` statement.
**`create procedure large_salaries`**
**`as`**
**`begin`**
	**`select * from employee_salary`**
	**`where salary >= 5000;`**
**`end;`**
**`go`** 

**`exec large_salaries;`**

Creating a stored procedure with more than one query.

**`create procedure large_salaries2`**
**`as`**
**`begin`**
	**`select * from employee_salary`**
	**`where salary >= 50000;`**
	**`select * from employee_salary`**
	**`where salary >= 1000;`**
**`end;`**

**`exec large_salaries2;`**

Creating stored procedures with parameters. `employee_id` is a parameter here.

**`create or alter procedure get_salary_from_employee_id`**
**`@employee_id int`**
**`as`**
**`begin`**
	**`select salary`**
	**`from employee_salary`**
	**`where employee_id = @employee_id;`**
**`end;`**

**`exec get_salary_from_employee_id @employee_id = 1;`**