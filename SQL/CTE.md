A Common Table Expression (CTE) in SQL is a temporary result set that can be referenced within a SELECT, INSERT, UPDATE, or DELETE statement. CTEs are defined using the WITH keyword and allow you to create a named, reusable subquery within your SQL statement. They provide a way to simplify complex queries and make them more readable.

`cte` stores the result of the inner query that is the select statement. Now we can access any aliases from the inner query. Remember that `cte` are to run immediately after creation. If you try to run it afterwards, it will give error saying table doesn't exist. You can even apply aggregation to the result of inner query, for example you can use `avg(avg_sal)` `avg_sal` comes from inner query and we are applying `avg()` over it.

**`with cte_example as`**
**`(`**
**`select`** 
	**`gender as gen,`** 
	**`avg(salary) as avg_sal,`** 
	**`max(salary) as max_sal,`** 
	**`min(salary) as min_sal,`** 
	**`count(salary) as count_sal`**
**`from employee_demographics as dem`**
**`join employee_salary as sal`**
	**`on dem.employee_id = sal.employee_id`**
**`group by gender`**
**`)`**
**`select *`** 
**`from cte_example`**
**`;`**


In case we want to apply conditions on two tables and merge their results. With the help of CTE we can do so.
**`with cte_example as`**
**`(`**
	**`select employee_id, gender, birth_date`**
	**`from employee_demographics`**
	**`where birth_date > '1985-01-01'`**
**`),`** 
**`cte_example2 as`**
**`(`**
	**`select employee_id, salary`**
	**`from employee_salary`**
	**`where salary > 50000`**
**`)`**
**`select *`** 
**`from cte_example`**
**`join cte_example2`** 
	**`on cte_example.employee_id = cte_example2.employee_id`**
**`;`**


Another way of creating Alias for CTE:
The alias that you have mentioned in the bracket beside `cte_example` will override the one that you have mentioned inside the `cte`. Now you can access the data using the alias given in brackets.
**`with cte_example (Gender, Avg_sal, Max_sal, Min_sal, Count_sal) as`**
**`(`**
**`select`** 
	**`gender as gen,`** 
	**`avg(salary) as avg_sal,`** 
	**`max(salary) as max_sal,`** 
	**`min(salary) as min_sal,`** 
	**`count(salary) as count_sal`**
**`from employee_demographics as dem`**
**`join employee_salary as sal`**
	**`on dem.employee_id = sal.employee_id`**
**`group by gender`**
**`)`**
**`select Avg_sal`**
**`from cte_example`**
**`;`**