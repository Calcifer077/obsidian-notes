Docstrings are a way to document your code. 
One way to add documentation is using comments using `#`.
```python
# this is a comment

num = 1 # this is another comment
```
Another way is to use **docstrings**. 

The utility of docstrings is that they follow conventions and as such they can be processed automatically.

Defining docstring for a function:
```python
def inc(n):
	"""Increments a number"""
	return n + 1;
```

Defining docstring for a class and a method:
```python
class Dog:
	"""A class representing a dog"""
	def __init__(self, name, age):
		"""Initialize a new dog"""
		self.name = name
		self.age = age
	
	def bark(self):
		"""Let the dog bark"""
		print('Wof')
```

Documenting a module, requires docstring at the top of the file:
```python
""" Dog module
This module does something and provides the following classes:

- Dog

"""
class Dog:
	"""A class representing a dog"""
	def __init__(self, name, age):
		"""Initialize a new dog"""
		self.name = name
		self.age = age
	
	def bark(self):
		"""Let the dog bark"""
		print('Wof')
```

Docstrings can span over multiple lines.

Python will process these docstrings and we can use `help()` global function to get the documentation for a class/method/function/module.

In case of IDE use `help()` in the code itself and in case of REPL you can use it in the terminal.

For example calling `help(inc)`  will give us this:
```text
Help on function inc in module __main__:

inc(n)
    Increment n by 1.
```
There are many standards out there and you can follow whichever one you like. One of the standards is by google.
[Github - google standard]([styleguide/pyguide.md at gh-pages · google/styleguide · GitHub](https://github.com/google/styleguide/blob/gh-pages/pyguide.md#38-comments-and-docstrings))
