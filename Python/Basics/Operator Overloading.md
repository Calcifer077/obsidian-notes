Operator overloading is an advanced technique we can use to make classes comparable and to make them work with Python operators.

Let's create a dog class:
```python
class Dog:
	def __init__(self, name, age):
		self.name = name
		self.age = age
```
Let's create two dogs:
```python
roger = Dog('roger', 9)
syd = Dog('Syd', 7)
```

We can use operator overloading to add a way to compare those 2 objects, say `age`
```python
class Dog:
	def __init__(self, name, age):
		self.name = name
		self.age = age
	
	def __gt__(self, other):
		return True if self.age > other.age else False
```

Now if you try running `print(roger > syd)` you will get the result `True`. You can basically add any amount of code in the above function, it just have to return `True` or `False`.

Example of more code:
```python
class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __gt__(self, other):
        if self.age == other.age:
            return self.name > other.name  # string comparison
        return self.age > other.age
```

In the same way we defined  `__gt__()`  (which means greater than), we can define the following methods:  
- `__eq__()`  to check for equality  
- `__lt__()`  to check if an object should be considered lower than another with the  `<`  operator  
- `__le__()`  for lower or equal ( `<=` )  
- `__ge__()`  for greater or equal ( `>=` )  
- `__ne__()`  for not equal ( `!=` )
Then you have methods to interoperate with arithmetic operations:
- `__add__()`  respond to the  `+`  operator  
- `__sub__()`  respond to the  `–`  operator  
- `__mul__()`  respond to the  `*`  operator  
- `__truediv__()`  respond to the  `/`  operator  
- `__floordiv__()`  respond to the  `//`  operator  
- `__mod__()`  respond to the  `%`  operator  
- `__pow__()`  respond to the  `**`  operator  
- `__rshift__()`  respond to the  `>>`  operator  
- `__lshift__()`  respond to the  `<<`  operator


