The `with` statement is a control-flow structure that simplifies resource management by automatically handling the setup and clean up of external resources such as files or network connections. It replaces cumbersome `try-except-finally` blocks, making the code cleaner, safer, and more readable.

Instead of writing:
```python
filename = '/Users/test.txt'

try:
	file = open(filename, 'r')
	content = file.read()
	print(content)
finally:
	file.close()
```
You can write:
```python
filename = '/Users/test.txt'

with open(filename, 'r') as file:
	content = file.read()
	print(content)
```

In other words we have built-in implicit exception handling.