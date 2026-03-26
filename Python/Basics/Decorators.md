Decorators are a way to change, enhance or alter in any way how a function works. In simply words, decorator is a function that takes another function as an argument, adds extra behaviour to it, and returns a new function, all without modifying the original function's source code.

A very simple example:
```python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        
        # 'func' is the argument, in this case it is `say_hello` function.
        func()

        print("Something is happening after the function is called.")

    return wrapper 

@my_decorator
def say_hello():
    print("Hello!")

say_hello();
```
In above code, first the statement before `func` will run, than `func` which is `say_hello` will run and than the statement after `func`.

Above decorator is same as writing:
```python
# If you don't use @ above 'say_hello'
main = my_decorator(say_hello)

main();
```

Whenever we call `say_hello`, decorator is going to be called.