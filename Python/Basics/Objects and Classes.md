Everything in Python is an object. Even values of basic primitive types (integer, string, float..) are objects. Lists, tuples, dictionaries, everything.

Objects have **attributes** and **methods** that can be accessed using the dot syntax.

Some objects are *mutable*, some are *immutable*. This depends on the object itself. If the object provides methods to changes its content, then it's mutable. Otherwise it's immutable. Most types defined by Python are immutable. For example an `int` is immutable. There are no methods to change its value. 

If you update any value using the methods provided by that object, its memory address (can be accessed using `id()`) will not be updated.

# Classes
In addition to using the Python-provided types, we can declare our own classes, and from classes we can instantiate object.

An object is an instance of a class. A class is the type of an object.

We can define a class in this way:
```python
class <class_name>:
	# your class
```
A class can define methods:
```python
class Dog:
	def bark(self):
		print('Woof!')
```
	`self` as the argument of the method points to the current object instance, and must be specified when defining a method.

How to create a instance of a class, a **object**:
```python
roger = Dog()
```
If you run
```python
print(type(roger))
```
You will get `<class '__main__.Dog'>`

A special type of method, `__init__()` is called constructor, and we can use it to initialize one or more properties when we create a new object from that class.
```python
class Dog:
	def __init__(self, name, age):
		self.name = name
		self.age = age
```
We can use it in this way:
```python
roger = Dog('Roger', 8)
print(roger.name) # 'Roger'
print(roger.age) # 8
```

### Inheritance
We can create an Animal class with a method `walk()`
```python
class Animal:
	def walf(self):
		print('Walking...')
```
and the Dog class can inherit from Animal:
```python
class Dog(Animal):
	def bark(self):
		print('Wof!')
```
Now creating a new object of class `Dog` will have the `walk()` method as that's inherited from `Animal`:
```python
roger = Dog() 
roger.walk() # 'Walking...' 
roger.bark() # 'Wof!'
```
