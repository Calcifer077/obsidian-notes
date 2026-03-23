Enums are readable names that are bound to a constant value (you can't modify them). 

To use enums, import `Enum` from the `enum` standard library module:
```python
from enum import Enum
```

Then you can initialize a new enum in this way:
```python
class State(Enum):
	INACTIVE = 0
	ACTIVE = 1
```
Once you do so, you can reference `State.INACTIVE` and `State.ACTIVE`, and they serve as constants.

Now if you try to print `State.ACTIVE` for example:
```python
print(State.ACTIVE)
```
it will not return `1`, but `State.ACTIVE`.
The same value can be reached by the number assigned in the enum: `print(State(1))`  will return `State.ACTIVE`. Same for using the square brackets notation `State['ACTIVE']` . 
You can however get the value using `State.ACTIVE.value`.

You can list all the possible values of an enum:
```python
list(State) # [<State.INACTIVE: 0>, <State.ACTIVE: 1>]
```

You can also count them:
```python
len(State) # 2
```

Python has no way to enforce a variable to be a constant. The nearest you can go is to use an enum:
```python
class Constants(Enum): 
	WIDTH = 1024 
	HEIGHT = 256
```
No one can reassign that value.