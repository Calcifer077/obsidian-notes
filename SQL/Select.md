
It is basically used to query columns from a database. You can even perform calculations in select statement.

`select` 
`first_name,` 
`last_name,` 
`age,`
`age + 10`
`from employee_demographics;`

In above code 'age + 10' will create a new column without any column name on it (in SQL server).

In sql order of Arithematic operations is as follows:
PEMDAS
P -> Paranthesis
E -> Exponentiation
M -> Multiplication
D -> Division
A -> Addition
S -> Subtraction

`select distinct gender from employee_demographics;`
selecting only distinct values from column.

`select distinct gender, first_name from employee_demographics;`
Now it treats gender, first_name as together meaning it will return the distinct combination of gender and first_name.