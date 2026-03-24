Every Python file is a module.
The main ideology behind this module is that we can import any python file into our main file. In a general case scenario there is a single entry point file in our application which imports functionalities from other files and provide that to the user.

Example of importing:
Suppose there is a `dog.py` containing following code:
```python
def bark():
	print('Woof!')
```
We can import this function in another file using `import`, and once we do, we can reference the function using the dot notation, `dog.bark()`:
```python
import dog

dog.bark()
```
Or, we can use the `from .. import` syntax and call the function directly:
```python
from dog import bark

bark()
```
The first strategy allows us to load everything defined in a file. 
The second strategy lets us pick the things we need.

Importing depends on the file system. Suppose you put `dog.py` in a `lib` subfolder.
In that folder, you need to create an empty file named `__init__.py`. This tells Python the folder contains modules. It is necessary for imports to work.
```python
from lib import dog

dog.bark();
```
or
```python
import lib.dog

lib.dog.bark();
```
or
```python
import lib.dog as dog # Using aliases

dog.bark()
```
or 
```python
from lib.dog import bark

bark()
```

Similarly you can use modules provided by Python Standard Library.
```python
import math

math.sqrt(4) # 2.0
```
or
```python
from math import sqrt

sqrt(4) # 2.0
```
Some modules by python:
- `math`  for math utilities 
- `re`  for regular expressions 
- `json`  to work with JSON 
- `datetime`  to work with dates 
- `sqlite3`  to use SQLite 
- `os`  for Operating System utilities 
- `random`  for random number generation 
- `statistics`  for statistics utilities 
- `requests`  to perform HTTP network requests 
- `http`  to create HTTP servers 
- `urllib`  to manage URLs