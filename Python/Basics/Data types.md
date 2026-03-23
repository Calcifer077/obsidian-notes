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

#### Booleans
`bool` type which can have two values: `True` and `False`.

When evaluating a value for `True` or `False`, if the value is not a `bool` we have some rules depending on the type we're checking:
- numbers are always `True` unless the number is `0`
- strings are `False` only when empty
- lists, tuples, sets, dictionaries are `False` only when empty.

You check if a value is a boolean in this way
```python
done = True
type(done) == bool # True

# OR

isinstance(done, bool) # True
```
The global `any()` function is also very useful when working with booleans, as it returns `True` if any of the values of the iterable (list, for example) passed as argument are `True`:
```python
book_1_read = True
book_2_read = False

read_any_book = any([book_1_read, book_2_read]) # True
```

The global `all()` function is same, but returns `True` if all the values passed to it are `True`:
```python
value1 = True
value2 = False

value3 = all([value1, value2]) # False
```

#### Numbers
Numbers is python can be of 3 types: `int`, `float` and `complex`
##### Integer number
Are represented using the `int` class. 
Ways to define:
```python
age = 8

age = int(10)
```

How to check type:
```python
type(age) == int # True
```

##### Floating point number
Fractions are of type `float`.
Ways to define:
```python
fraction = 0.1

fraction = float(0.2)
```

How to check type:
```python
type(fraction) == float # True
```

##### Complex number
type `complex`.
Ways to define:
```python
complexNumber = 2 + 3j # You need to use `j`

complexNumber = complext(2, 3);
```
How to get its real and imaginary part:
```python
complexNumber.real # 2.0
complexNumber.imag # 3.0
```

##### Arithmetic operations
`**` for exponentiation, `//` for floor division, rest are same as in other languages