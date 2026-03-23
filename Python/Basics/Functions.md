Syntax for creating a function:
```python
def hello():
	print('Hello!')
```
Above is called function **definition**. To run this we use `hello()`. 

Passing parameters:
```python
def hello(name = 'roger', age):
	print('Hello ' + name + '-' + str(ge) + '!);
```
Above `name` is a default parameter, `roger` will be used when you don't provide anything.
	We call parameters the values accepted by the function inside the function definition, and arguments the values we pass to the function when we call it. It's common to get confused about this distinction.

Parameters are passed by reference. All types in Python are objects but some of them are immutable, including integers, booleans, floats, strings, and tuples. This means that if you pass them as parameters and you modify their value inside the function, the new value is not reflected outside of the function.
```python
def change(value): 
	value = 2 
	
val = 1 
change(val) 
print(val) #1
```
If you pass an object that's not immutable, and you change one of its properties, the change will be reflected outside.

You can return multiple values from a function
```python
def hello(name):
	return name, 'Roger', 9
```
What you return will be in a tuple form.