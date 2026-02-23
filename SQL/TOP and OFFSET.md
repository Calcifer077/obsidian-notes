The `TOP` clause is used to retrieve a specified number or percentage of rows from the result set of a `SELECT` statement.

`select top 3 * from employee_demographics;`

gets top 3 rows from the result.

`OFFSET-FETCH` is used for implementing pagination, allowing you to skip a certain number of rows and then fetch a specified number of subsequent rows. It must be used with an `ORDER BY` clause.
`select *` 
`from employee_demographics`
`order by employee_id`
`offset 1 rows`
`fetch next 3 rows only;`

### Step-by-step explanation:

1. **`ORDER BY employee_id`**
    
    - Sorts all rows in the table by `employee_id` (ascending by default).
        
    - This ensures that the “first row” and the “next rows” are well-defined.
        
2. **`OFFSET 1 ROWS`**
    
    - Skips the first row of the ordered result set.
        
    - If the lowest `employee_id` is 1, that row will be skipped.
        
3. **`FETCH NEXT 3 ROWS ONLY`**
    
    - After skipping 1 row, it pulls the **next 3 rows**.
        
    - So, rows 2, 3, and 4 (based on the sorted order) will be returned.