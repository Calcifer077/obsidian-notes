Dictionary allows us to create collections of **key / value pairs** just like **hashmap** in other languages.

Example with key/value pair:
```python
dog = {'name': 'Roger', 'age': 8}
```
The key can be any immutable value like a string, a number or a tuple. Same goes for value.

You can even do and this will give no error:
```python
a = {'a': 'b', 1: '2'}
```
You can access individual key values using below notation:
```python
dog['name'] # 'Roger'
dog['age'] # 8
```
You can also modify using the above notation:
```python
dog['name'] = 'Syd'
```
Another way of getting a value in case the key doesn't exist is:
```python
dog.get('name') # 'Roger'
dog.get('test', 'default') # 'default'
```
The above code will return `default` as `test` doesn't exist in dictionary. If you didn't use a default value you will get `None`.

The `pop()` method retrieves the value of a key, and removes it from the dictionary.
```python
dog.pop('name')
```

The `popitem()` method retrieves and removes the last key / value pair inserted into the dictionary:
```python
dog.popitem()
```
You can check if a key is contained into a dictionary with the `in` operator:
```python
'name' in dog # True
```
Get a list with keys in dictionary using the `keys()` method, passing its result to the `list()` constructor:
```python
list(dog.keys()) # ['name', 'age']
```
Get the values using the `values()` method, and the key/value pairs tuple using the `items()` method:
```python
print(list(dog.values()))
# ['Roger', 8]

print(list(dog.items()))
# [('name', 'Roger'), ('age', 8)]
```
You can add a new key/value pair to the dictionary in this way:
```python
dog['favorite food'] = 'Meat'
```
You can remove a key/value pair from a dictionary using `del`:
```python
del dog['favorite food']
```
To copy a dictionary, use the `copy()` method:
```python
dogCopy = dog.copy()
```


