Getting length:
**`select len('mahesh');`**

**`select first_name, len(first_name) as [Employee name length]`**
**`from employee_demographics;`**

Converts to upper case
**`select upper('sky');`**
Converts to lower case
**`select lower('sKy');`**

Trim -> Removes whitespaces
Removes from both ends
**`select trim('    dk ');`**

Removes from right side only
**`select rtrim('    dk   ');`**

Removes from left side only
**`select ltrim('    dk    ');`**

left, goes from the left hand side and only prints the specified amount of characters same is true for right, it just goes from right hand side.
**`select first_name, left(first_name, 4), right(first_name, 4)`**
**`from employee_demographics;`**

Substring -> where to start and how many characters to take.
**`select first_name, substring(first_name, 3, 2)`**
**`from employee_demographics;`**

Replace -> replaces certain characters with some other characters. Is case sensitive.
**`select first_name, replace(first_name, 'a', 'z')`**
**`from employee_demographics;`**

Charindex -> searches for a expression(can be of more than one word). Returns the first place where it found it. Third parameter is optional, it tells where to start searching for. Index is 1 based.
**`select charindex('x', 'Alexxxander');`**
**`select charindex('x', 'Alexxxander', 12);`**

Concat -> joins multiples strings, can be any number of strings.
**`select first_name, last_name,`**
**`concat(first_name, ' ', last_name)`**
**`from employee_demographics;`**