##### 1. Assignment Operator
Used to assign a value to variable. `=`

##### 2. Arithmetic Operator
##### 3. Comparison Operator
##### 4. Boolean Operator
- `not`
- `and`
- `or`
```python
condition1 = True 
condition2 = False 
not condition1 #False 
condition1 and condition2 #False 
condition1 or condition2 #True
```

`or` used in an expression returns the value of the first operand that is not a falsy value (`False`, `0`, `''`, `[]`). Otherwise it returns the last operand.
```python
print(0 or 1) ### 1 
print(False or 'hey') ### 'hey' 
print('hi' or 'hey') ### 'hi' 
print([] or False) ### False 
print(False or []) ### []
```
Python describe it as `if x is false, then y, else x`.

`and` only evaluates the second argument if the first one is true. So if the first argument is falsy (`False`, `0`, `''`, `[]`), it returns that argument. Otherwise it evaluates the second argument:
```python
print(0 and 1) ### 0 
print(1 and 0) ### 0 
print(False and 'hey') ### False 
print('hi' and 'hey') ### 'hey' 
print([] and False ) ### [] 
print(False and [] ) ### False
```
Python describe it as `if x is false, then x, else y`.

##### 5. Bitwise operator
Some operators are used to work on bits and binary numbers:
- `&` performs binary AND
- `|` performs binary OR
- `^` performs binary XOR 
- `~` performs binary NOT 
- `<<` performs shift left operation
- `>>` performs shift right operation

##### 6. `is` and `in`
`is` is called the **identity operator**. It is used to compare two objects and returns true if both are the same object.

`in` is called the **membership operator**. Is used to tell if a value is contained in a list, or another sequence.
```python
name = 'Roger'
print('ger' in name) # True
```