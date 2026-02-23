A temporary table is a temporary storage area within a database session designed to hold intermediate results or data during complex queries or procedures. When you create a temp table you can directly see them in the database schema, they exist in the memory and will continue to do so until the session is alive(until you don't close the window).

Ways to create a temporary table:
This is a local temporary table and only visible to the current session (the one who created it)  and automatically dropped when the session ends.
**`create table #temp_table (`**
	**`first_name varchar(50),`**
	**`last_name varchar(50),`**
	**`favorite_movie varchar(100)`**
**`);`**

**`select * from #temp_table;`**

Global temporary table -> visible to all sessions, dropped automatically whent he session that created them ends and no other sessions are referencing them.
**`create table ##global_temp_table(`**
	**`first_name varchar(50),`**
	**`last_name varchar(50),`**
	**`favorite_movie varchar(100)`**
**`);`**

You can do all operations on the temporary tables.
**`insert into #temp_table values`**
**`('Alex', 'Freberg', 'LOTR');`**

Another way of creating a temp table, in this case we are basically getting all the values from one table and inserting them into another table. If you want to have control on what values should come into your table, than declare your table structure first hand and than do the insertion.
**`select *`** 
**`into #salary_over_50k`**
**`from employee_salary`**
**`where salary >= 50000;`**

**`select * from #salary_over_50k;`**