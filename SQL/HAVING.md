The `HAVING` clause was added to SQL because the `WHERE` keyword cannot be used with aggregate functions.

Why 'WHERE'  can't be used with group by. Because 'WHERE' is used to filter rows before they are grouped.

`select gender, avg(age)` 
`from employee_demographics` 
`where avg(age) > 40`
`group by gender;`

We are using 'WHERE' before 'group by' to get only those who have 'avg(age) > 40' but grouping haven't been done yet.

Solution:
`select gender, avg(age)` 
`from employee_demographics` 
`group by gender`
`having avg(age) > 40;`

Using both 'where' and 'having':
`select occupation, avg(salary)`
`from employee_salary`
`where occupation like '%manager'`
`group by occupation`
`having avg(salary) > 60000;`

Firstly we filter out rows, than use filter on aggregate function.