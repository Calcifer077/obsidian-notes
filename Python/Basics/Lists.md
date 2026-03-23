DS in Python. They allow us to group together multiple values and reference them all with a common name.
Example:
```python
dogs = ['Roger', 'Syd']
```
A list can hold values of different types:
```python
items = ['Roger', 1, 'Syd', 0]
```

You can check if an item is present in a list or not using `in`
```python
print('Roger' in items) # True
```
#### Accessing
You can reference items in a list by their index. You can also assign them values using indexes. Can also use negative index, will start searching from back.
```python
items[0] # 'Roger'
items[1] # 1
items[2] = 'Maria'
```
Above if you try to access  out of range you will get error.

How can we get a index of a item?
```python
items.index('Roger') # 0
items.index('Syd') # 2
```

You can also extract a part of a list, using slices:
```python
items[0:2] # ['Roger', 1]
items[2:] # ['Syd', 0]
```

#### Adding and Removing
How to add items to a list?
```python
items.append('Test')
items.extend(['Test'])
items += ['Test']

# Add multiple items
items += ['Test1', 'Test2']
# or
items.extend(['Test1', 'Test2'])
```
	Note: with  extend()  or  +=  don't forget the square brackets. Don't do  items += "Test"  or  items.extend("Test")  or Python will add 4 individual characters to the list, resulting in ['Roger', 1, 'Syd', True, 'T', 'e', 's', 't']

You can remove using `remove()`. Pass the item that you want to remove as it is. If there are multiple occurrences will remove the first occurrence.

To add an item in the middle of a list, or at a specified index, use `insert()` method:
```python
items.insert(1, 'test') # will shift everything to right

# To add multiple items
items[1:1] = ['test', 'test', 'test'] 
# '1' is the index where to add, this way you can add any number of items and all other will be shifted to the right.
```

#### Sorting
We can sort using `sort()` method.
```python
items.sort()
```
	Note: `sort()` will only work if the list holds value that can be compared. Should be of similar data types else you will get a error.
The `sort()` method order uppercase first, then lowercased. To fix this, use:
```python
items.sort(key = str.lower)
```
Sorting modifies the original list. To avoid that, you can copy the list content using 
```python
itemscopy = items[:]
```
or use the `sorted()` global function:
```python
print(sorted(items, key = str.lower))
```
that will return a new list, sorted, instead of modifying the original list.