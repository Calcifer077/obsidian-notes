Python have several built in types. You just need to assign a value to a variable and the language will automatically detect the type.

```python
name = "Roger" # this is type string
```

How to check for type of a variable:
```python
name = "Roger"
type(name) == str # True
```
Or using `isinstance`
```python
name = "Roger"
isinstance(name, str) # True
```

Similarly there are other types:
```python
age = 1 # int
fraction = 1.1 # float
```

You can also create a variable of a specific type by using class constructor.
```python
name = str("Flavio") 
anotherName = str(name)
```

You can also convert from one datatype to another using class constructor. Will only work if valid value otherwise you will get a `ValueError`. This process is known as casting.
```python
age = int("20") 
print(age) #20 
fraction = 0.1 
intFraction = int(fraction) # will always floor the value of fraction
print(intFraction) # 0
```

Other data types in python:
* `complex` for complex numbers
* `bool` for booleans
* `list` for lists
* `tuple` for tuples
* `range` for ranges
* `dict` for dictionaries
* `set` for sets