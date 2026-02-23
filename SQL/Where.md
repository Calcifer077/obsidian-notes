
It is used to apply condition to our select statement.
`select * from employee_demographics where first_name = 'Leslie';`

Here, '=' is a comparison operator.

`select * from employee_demographics where gender != 'Female';`
Here, '!=' means not equal to.

comparison with dates
`select * from employee_demographics where birth_date > '1987-03-04';`

Format of date: 'YEAR-MONTH-DAY'. We can apply basic operations on date.

AND, OR, NOT -> Logical operators
`select * from employee_demographics where not gender = 'Male';`
Use case of 'not', will return all females.