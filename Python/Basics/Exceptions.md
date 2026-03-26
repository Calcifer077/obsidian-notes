To handle exceptions in Python we can use `try ... except` block. `except` is similar to `catch` in other languages.

```python
try:
	# Some lines of code
except <ERROR1>:
	# hanlder <ERROR1>
except <ERROR2>:
	# hanler <ERROR2>
except:
	# catch all other exceptions
else:
	# no excpetions were raised, the code ran successfully
finally:
	# do something in any case
```
If an error occurs, Python will alert us and we can determine which kind of error occurred using a `except` block.
To catch all exceptions we can use `except` without any error type.
The `else` block is ran if no exceptions were found.
A `finally` block lets you perform some operation in any case, regardless if an error occurred or not.

The specific error that's going to occur depends on the operation you're performing.

A very simple example:
```python
try:
	result = 2 / 0
except ZeroDivisionError:
	print('Cannot divide by zero')
finally:
	result = 1
	
print(result) # 1
```

You can raise exceptions in your code, by using `raise` statement.
```python
raise Exception('An Error Occured')
```

Usage:
```python
try:
	raise Exception('An error occured')
except Exception as error:
	print(error) # 'An error occured'
```

You can also define your own exception class, extending from Exception:
```python
class DogNotFoundExcpetion(Exception):
	pass
```
	`pass` here means 'nothing' and we must use it when we define a class without methods, or a function without code, too.

Usage:
```python
try:
	raise DogNotFoundException()
except DogNotFoundException:
	print('Dog not found')
```

Exceptions follow a order, meaning that if in the `try` block there occur any exception nothing below it will run and its corresponding `except` block will run and than `finally`.

[Built-in Exceptions](https://docs.python.org/3/library/exceptions.html)
