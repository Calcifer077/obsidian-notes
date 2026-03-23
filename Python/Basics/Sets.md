Stores unique values without ordering. Hence they are mutable.

They also have an immutable version, called `frozenset`.

Creating a set:
```python
names = {'Roger', 'Syd'}
```
You can intersect two sets:
```python
set1 = {'Roger', 'Syd'}
set2 = {'Roger'}

intersect = set1 & set2 # {'Roger'}
```

Union of two sets:
```python
set1 = {'Roger', 'Syd'}
set2 = {'Luna'}

union = set1 | set2 # {'Syd', 'Luna', 'Roger'}
```
You can get the difference between two sets:
```python
set1 = {"Roger", "Syd"} 
set2 = {"Roger"} 
difference = set1 - set2 # {'Syd'}
```
You can check if a set is a superset of another:
```python
set1 = {"Roger", "Syd"} 
set2 = {"Roger"} 
isSuperset = set1 > set2 # True
```
You can get a list from the items in a set by passing the set to the `list()` constructor:
```python
names = {'Roger', 'Syd'}
list(name) # ['Roger', 'Syd']
```
You can check if an item is contained into a set with the `in` operator:
```python
print('Roger' in names) # True
```
