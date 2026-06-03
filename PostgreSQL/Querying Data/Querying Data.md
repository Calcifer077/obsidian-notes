Backlinks with summary of topics:
- [SELECT](SELECT.md)
## Summary of [SELECT](SELECT.md)

- Use the `SELECT ... FROM` statement to retrieve data from a table.
- PostgreSQL evaluates the `FROM` clause before the `SELECT` clause.
- Use a column alias to assign a temporary name to a column or an expression in a query.
- In PostgreSQL, the `FROM` clause is optional.

- [Column alias](Column%20alias.md)
## Summary of [Column alias](Column%20alias.md)

- Assign a column or an expression a column alias using the syntax `column_name AS alias_name` or `expression AS alias_name`. The `AS` keyword is optional.
- Use double quotes (“) to surround column aliases that contain spaces.

- [ORDER BY](ORDER%20BY.md)

## Summary of [ORDER BY](ORDER%20BY.md)

- Use the `ORDER BY` clause in the `SELECT` statement to sort the rows in the query set.
- Use the `ASC` option to sort rows in ascending order and `DESC` option to sort rows in descending order.
- The `ORDER BY` clause uses the `ASC` option by default.
- Use `NULLS FIRST` and `NULLS LAST` options to explicitly specify the order of `NULL` with other non-null values.

- [SELECT DISTINCT](SELECT%20DISTINCT.md)
## Summary of [SELECT DISTINCT](SELECT%20DISTINCT.md)

- Use the `SELECT DISTINCT` to remove duplicate rows from a result set of a query.

- [DISTINCT ON](DISTINCT%20ON.md)
## Summary of [DISTINCT ON](DISTINCT%20ON.md)

- Use the `DISTINCT ON` clause to keep the first unique entry from each column or combination of columns in a result set.
- Always use the `ORDER BY` clause to determine which entry to retain in the result set.




