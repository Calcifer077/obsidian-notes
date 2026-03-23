Fundamental DS in Python.
Allows you to create immutable group of objects. This means that once a tuple is created, it can't be modified. You can't add or remove items.

They are created in a way similar to lists, but using parentheses instead of square brackets.
```python
name = ('Roger', 'Syd')
```

A tuple is ordered, like a list, so you can get its values referencing an index value, you can also use `index()` method, use negative indexes to access, use `in` operator, use `slicing`, `sorting`. Just the main difference between tuple and list is that tuple are immutable.

You can create a new tuple from existing tuples using `+` operator:
```python
newtuple = names + ('Vanilla', 'Tina')
```
