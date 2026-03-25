Lambda function (also called anonymous functions) are functions that have no name and only have one expression as their body. They must return something (does on their own, you don't need to `return` anything.

Syntax:
```python
lambda <arguments> : <expression>
```
The body must be a single expression. Expression, not a statement.
	This difference is important. An expression returns a value, a statement does not.

The simplest example of a lambda function is a function that doubles that value of a number:
```python
lambda num : num * 2
```

Can accept more arguments:
```python
lambda a, b : a * b
```

Using **lambda** functions:
```python
multiply = lambda a, b : a * b

print(multiply(2, 2)) # 4
```
