Python has its inbuilt debugger called `pdb` which can be invoked without importing anything.

You can start debugging by adding breakpoint in your code:
```python
breakpoint()
```
	You can add any number of breakpoints according to your needs.

When the Python interpreter hits a breakpoint in our code, it will stop, and it will tell us what is the next instruction it will run.

You don't need any special commands to invoke `pdb`. It will be invoked by simply `python <file_name>.py`.

Once you have debugger running, it will ask for your commands to execute. Some of the commands are listed below:
- `continue` - `c` - Continue execution until the next breakpoint in hit.
- `next` - `n` - Execute the current line and stop at the next line within the same function.
- `step` - `s` - Step into a function call on the current line
- `list` - `l` - List the source code around the current line of execution
- `print` - `p` - Evaluate and print the value of an expression or value in the current context. Use: `p <variable_name>`
- `quit` - `q` - Exit the debugger and terminate the program
- `help` - `h` - Display list of commands.
