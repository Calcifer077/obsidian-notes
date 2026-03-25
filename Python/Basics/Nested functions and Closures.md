You can define function inside a function in python. A function defined inside a function is visible only inside that function. This is useful to create utilities that are useful to a function, but not useful outside of it.

```python
def talk(phrase):
	def say(word):
		print(word)
		
	words = phrase.split('')
	for word in words:
		say(word)
		
talk('I am going to buy milk')
```
If you want to access a variable defined in the outer function from the inner function, you first need to declare it as `nonlocal`.
```python
def count():
	count = 0
	
	def inc():
		nonlocal count
		count = count + 1
		print(count)
		
	inc()
	
count()
```
`count` calls `inc` inside it and we call `count` in the global scope.

# Closures
If you return a nested function from a function, that nested function has access to the variables defined in that function, even if that function is not active any more.

Example:
```python
def counter():
	count = 0
	
	def inc():
		nonlocal count
		count = count + 1
		return count
		
	return inc
	
inc = counter()

print(inc()) # 1
print(inc()) # 2
print(inc()) # 3
```
We return the `inc()` inner function, and that has still access to the state of the `count` variable even though the `counter()` function has ended.