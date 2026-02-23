The `GROUP BY` statement groups rows that have the same values into summary rows, like "find the number of customers in each country".

The `GROUP BY` statement is often used with aggregate functions (`COUNT()`, `MAX()`, `MIN()`, `SUM()`, `AVG()`) to group the result-set by one or more columns.

**`select gender, avg(age) from employee_demographics group by gender;`**
It groups all rows into groups based on gender. Than we have used a aggregate function 'avg' which will give average age of all employee that belongs to that particular group.

If you are not performing a aggregate function on any column in the select statement, than that column needs to be present in the group by clause.