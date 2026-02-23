Used in where clause to search for specific pattern in a column.

There are two wildcards often used in conjunction with the `LIKE` operator:

- The percent sign `%` represents zero, one, or multiple characters
- The underscore sign `_` represents one, single character

`select * from employee_demographics where first_name like 'jer%';`
All employee whose first_name has 'jer' in the start.

`select * from employee_demographics where first_name like '%er%';`
All employee who have 'er' somewhere in their first_name, can be anywhere, at first position, last position or in the middle.

`select * from employee_demographics where first_name like 'a__';`
All employee whose first_name starts with 'a' and there are exactly two characters after it, no matter what, but exactly two.

Combining '_ ' and '%':
`select * from employee_demographics where first_name like 'a__%';`

All employee who have a 'a' in the start and then exactly two characters than any number of characters(can be zero).

Also works with dates:
`select * from employee_demographics where birth_date like '1989%';`