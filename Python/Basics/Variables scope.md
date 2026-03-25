When you declare a variable, that variable is visible in parts of your program, depending on where you declare it.

**Global Variable**
If you declare it outside of any function, the variable is visible to any code running after the declaration, including functions:
```python
age = 8

def test():
	print(age)
	
print(age) # 8
test() # 8
```
If you try to access variable before declaration you will get error. In case of functions the variable should be declared before calling the function.
```python
print(a) # Error

a = 10;

def test():
	print(age)
	
age = 10
test() # 10
```

**Local Variable**
If you define a variable inside a function, that variable is a local variable, and it is only visible inside that function. Outside the function, it is not reachable.
```python
def test():
	age = 9
	print(age)
	
test() # 9

print(age) # NameError: name 'age' is not defined
```
