In the most basic way when you run a python file you do:
```bash
python <filename>.py
```

You can pass additional arguments and options in the above command.
```bash
python <filename>.py <argument1>
python <filename>.py <argument1> <argument2>
```
A basic way to handle these arguments is to use `sys` module from the standard library.
```python
import sys

print(sys.argv)
```
The `sys.argv` list contains as the first item the name of the file that was run, e.g. `['main.py']`

Above is a simple way, but you have to do a lot of work. You need to validate arguments, make sure their type is correct, you need to print feedback to the user if they are not using the program correctly.

Python provides another package in the standard library to help us: `argparse`.

Usage:
```python
import argparse

parser = argparse.ArgumentParser(
	description='This program accepts a color'
)

parser.add_argument('-c', '--color', metavar='color', required=True, help='A color name')

args = parser.parse_args()

print(args.color)
```
`-c` or `--color` can be used to pass the arguments to our program. `metavar` is used when there is error say user didn't provide color. It will show user the correct usage and use the `matavar` variable in that usage. `help` is also printed in case of error.

```text
python python program.py 
usage: program.py [-h] -c color 
program.py: error: the following arguments are required: -c/--color
```

You can set an option to have a specific set of values, using `choices`:
```python
parser.add_argument('-c', '--color', metavar='color', required=True, choices={'red', 'yellow'}, help='A color name\)
```
```text
python python program.py -c blue
usage: program.py [-h] -c color
program.py: error: argument -c/--color: invalid choice: 'blue'
(choose from 'yellow', 'red')
```
