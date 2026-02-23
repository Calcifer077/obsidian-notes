Basically allows us to add if/else in sql.

case allows us to add multiple if statements. the first one that is correct is executed.
**`select first_name, last_name, age,`**
**`case`**
	**`when age <= 30 then 'Young'`**
	**`when age between 31 and 50 then 'Old'`**
	**`when age >= 50 then 'too old'`**
**`end as age_bracket`**
**`from employee_demographics`**
**`;`**


We can add multiple case statements.
**`select first_name, last_name, salary,`**
**`case`**
	**`when salary < 50000 then salary * 1.05`**
	**`when salary > 50000 then salary * 1.07`**
**`end as new_salary,`**
**`case`**
	**`when dept_id = 6 then salary * .10`**
**`end as bonus`**
**`from employee_salary;`**

If some row doesn't match any case statements than null will be displayed in the resulting column.
If you don't want null to appear, you can use a else statement, but keep in mind that the datatype should be same to rest of the case statements.
**`select first_name, last_name, salary,`**
**`case`**
	**`when salary < 50000 then salary * 1.05`**
	**`when salary > 50000 then salary * 1.07`**
	**`else 0`**
**`end as new_salary,`**
**`case`**
	**`when dept_id = 6 then salary * .10`**
	**`else 0`**
**`end as bonus`**
**`from employee_salary;`**