SQL window functions allow performing calculations across a set of rows that are related to the current row, without collapsing the result into a single value. They are commonly used for tasks like aggregates, rankings and running totals.

Window is kind of similar to group by. How they differ:
group by -> aggregates rows into groups. Returns one row per group. Collapses the data — you lose row-level detail.
window -> Perform calculations **across a set of rows** (the “window”) without collapsing them. Returns a value **for each row**. Preserves row-level detail while adding aggregate-like info.

Below is a example of average by gender. So it will return two rows one for male and another for female and show their respective average salary beside them. If you were to need more details like names of employee than you would have to add those into group by which will create unique combinations of `gender`, `first_name`, `last_name` resulting in not getting average salary based on gender.
**`select gender, avg(salary)`**
**`from employee_demographics dem`**
**`join employee_salary sal`**
	**`on dem.employee_id = sal.employee_id`**
**`group by gender`**	
**`;`**

Below is a example of doing the same thing as above using window function. `partition` tells us how we should divide our result set. It basically divides them into groups, in this case based on `gender`. If you didn't specify `partition` it will treat each row as a separate group.
**`select`** 
	**`dem.first_name,`** 
	**`dem.last_name,`** 
	**`gender,`** 
	**`avg(salary) over(partition by gender)`**
**`from employee_demographics dem`**
**`join employee_salary sal`**
	**`on dem.employee_id = sal.employee_id`**
**`;`**


Below we have done something known as a `rolling total`, what it means is it starts from the first value and adds up the next value and so on until a certain group continues. Kind of like prefix sum. In below case we first divided into two groups by gender and took the sum of their salary like the second one is the sum of first and second and so on.
**`select`** 
	**`dem.first_name,`** 
	**`dem.last_name,`** 
	**`gender,`** 
	**`salary,`**
	**`sum(salary) over(partition by gender order by dem.employee_id) as rolling_total`**
**`from employee_demographics dem`**
**`join employee_salary sal`**
	**`on dem.employee_id = sal.employee_id`**
**`;`**

**`row_number`** -> gives a unique row number to resulting rows starting from 1 with a increment of 1. when using `row_number` in ssms, `order by` is required.
**`select`** 
	**`dem.first_name,`** 
	**`dem.last_name,`** 
	**`gender,`** 
	**`salary,`**
	**`row_number() over(order by dem.employee_id) as row_num`**
**`from employee_demographics dem`**
**`join employee_salary sal`**
	**`on dem.employee_id = sal.employee_id`**
**`;`**

This is the same as above but this time we have created a partition, meaning there are two groups now. So `row_number` will start for one group `female` and end and than again start for `male`. In both case will start from 1.
**`select`** 
	**`dem.first_name,`** 
	**`dem.last_name,`** 
	**`gender,`** 
	**`salary,`**
	**`row_number() over(partition by gender order by dem.employee_id) as row_num`**
**`from employee_demographics dem`**
**`join employee_salary sal`**
	**`on dem.employee_id = sal.employee_id`**
**`;`**

**`rank`** -> kind of similar to `row_number` but it in this case if two values which we have `order by `over have same value than they will get the same rank. If two values are same then it will give them the same rank and than skip.
`dense_rank` -> similar to 'rank' it also give same rank to duplicates but doesn't skip rank ordering.
for example:
salary row_number      rank     dense_rank
100       1                        1              1
90         2                        2              2 
90         3                        2              2
80         4                        4              3

**`select`** 
	**`dem.first_name,`** 
	**`dem.last_name,`** 
	**`gender,`** 
	**`salary,`**
	**`row_number() over(partition by gender order by salary desc) as row_num,`**
	**`rank() over(partition by gender order by salary desc) as row_rank,`**
	**`dense_rank() over(partition by gender order by salary desc) as row_dense_rank`**
**`from employee_demographics dem`**
**`join employee_salary sal`**
	**`on dem.employee_id = sal.employee_id`**
**`;`**

