The `ORDER BY` keyword is used to sort the result-set in ascending or descending order.

Can also work on string. Default is Ascending (ASC), you can change it to Descending (DESC).

`select * from employee_demographics order by first_name ASC;`

You can also use more than one column in order by.

`select * from employee_demographics order by gender, age;`

First it will sort by gender, than by age. You can even apply ascending descending to individual column. For above ordering to work, there should be some similar gender, if all gender were different than it would only sort using gender, but if there were some same gender, than it would sort using age for those.

It is not recommended but you can also use order by on column number.

`select * from employee_demographics order by 5, 4;`

Indexing starts from 1.