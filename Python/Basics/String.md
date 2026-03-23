A string in is a series of characters enclosed into single or double quotes:
```python
name1 = 'Roger'
name2 = "Roger"
```
You can concatenate using `+` and caste using `str()`.

A string can be multiline if you enclose it with set of 3 quotes. Single or double.
```python
#double quotes, or single quotes 
print("""Roger 
is 8 

years 
old """) 

print(''' 
Roger is 8 
years old
''')
```
Inbuilt methods of string:
- `isalpha()`  to check if a string contains only characters and is not empty 
- `isalnum()`  to check if a string contains characters or digits and is not empty 
- `isdecimal()`  to check if a string contains digits and is not empty 
- `lower()`  to get a lowercase version of a string 
- `islower()`  to check if a string is lowercase 
- `upper()`  to get an uppercase version of a string 
- `isupper()`  to check if a string is uppercase 
- `title()`  to get a capitalized version of a string 
- `startswith()`  to check if the string starts with a specific substring
- `endswith()`  to check if the string ends with a specific substring 
- `replace()`  to replace a part of a string 
- `split()`  to split a string on a specific character separator 
- `strip()`  to trim the whitespace from a string 
- `join()`  to append new letters to a string 
- `find()`  to find the position of a substring
- `len()` gives length of a string

None of these methods alter the original string. They return a new, modified string instead.

##### Escaping
It is a way to add special characters into a string. For example, how do you add a double quote into a string that's wrapped into double quotes?
`"Ro"ger"` will not work, as, Python will think the string ends at `"Ro"`.

The way to go is to escape the double quite inside the string, with `\` backslash character.
```python
name = "Ro\"ger"
```
This applies to single quotes too `\'`, and for special formatting characters like `\t` for tab, `\n` for new line and `\\` for backslash.

You can access a string characters using square brackets. Index starts from 0.
```python
name = "Roger"
name[0] # 'R'
name[1] # 'o'
name[2] # 'g'
```

Using a negative number will start counting from the end:
```python
name = "Roger"
name[-1] # 'r'
```

You can also use a range, using what we call **slicing**:
```python
name = 'Roger'
name[0:2] # 'Ro'
name[:2] # 'Ro' 
name[2:] # 'ger'
```