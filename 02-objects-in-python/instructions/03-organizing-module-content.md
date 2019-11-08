Inside any one module, we can specify  variables, classes, or functions. They can be a handy way to store the  global state without namespace conflicts. For example, we have been  importing the `Database` class into various modules and then instantiating it, but it might make more sense to have only one `database` object globally available from the `database` module. The `database` module might look like this:

```python
class Database: 
    # the database implementation 
    pass 
 
database = Database() 
```

Then we can use any of the import methods we've discussed to access the `database` object, for example:

```python
from ecommerce.database import database 
```

A problem with the preceding class is that the `database`  object is created immediately when the module is first imported, which  is usually when the program starts up. This isn't always ideal, since  connecting to a database can take a while, slowing down startup, or the  database connection information may not yet be available. We could delay  creating the database until it is actually needed by calling an `initialize_database` function to create a module-level variable:

```python
class Database: 
    # the database implementation 
    pass 
 
database = None 
 
def initialize_database(): 
    global database 
    database = Database() 
```

The `global` keyword tells Python that the database variable inside `initialize_database`  is the module level one we just defined. If we had not specified the  variable as global, Python would have created a new local variable that  would be discarded when the method exits, leaving the module-level value  unchanged.

As these two examples illustrate, all module-level  code is executed immediately at the time it is imported. However, if it  is inside a method or function, the function will be created, but its  internal code will not be executed until the function is called. This  can be a tricky thing for scripts that perform execution (such as the  main script in our e-commerce example). Sometimes, we write a program  that does something useful, and then later find that we want to import a  function or class from that module into a different program. However,  as soon as we import it, any code at the module level is immediately  executed. If we are not careful, we can end up running the first program  when we really only meant to access a couple of functions inside that  module.

To solve this, we should always put our start up code in a function (conventionally, called `main`)  and only execute that function when we know we are running the module  as a script, but not when our code is being imported from a different  script. We can do this by **guarding** the call to `main` inside a conditional statement, demonstrated as follows:

```python
class UsefulClass:
    """This class might be useful to other modules."""

    pass


def main():
    """Creates a useful class and does something with it for our module."""
    useful = UsefulClass()
    print(useful)


if __name__ == "__main__":
    main()
```

Every module has a `__name__` special variable (remember, Python uses double underscores for special variables, such as a class's `__init__` method) that specifies the name of the module when it was imported. When the module is executed directly with `python module.py`, it is never imported, so the `__name__` is arbitrarily set to the `"__main__"` string. Make it a policy to wrap all your scripts in an `if __name__ == "__main__":` test, just in case you write a function that you may want to be imported by other code at some point in the future.

So, methods go in classes, which go in modules, which go in packages. Is that all there is to it?

Actually,  no. This is the typical order of things in a Python program, but it's  not the only possible layout. Classes can be defined anywhere. They are  typically defined at the module level, but they can also be defined  inside a function or method, like this:

```python
def format_string(string, formatter=None):
    """Format a string using the formatter object, which 
    is expected to have a format() method that accepts 
    a string."""

    class DefaultFormatter:
        """Format a string in title case."""

        def format(self, string):
            return str(string).title()

    if not formatter:
        formatter = DefaultFormatter()

    return formatter.format(string)


hello_string = "hello world, how are you today?"
print(" input: " + hello_string)
print("output: " + format_string(hello_string))
```

The output would be as follows:

```reStructuredText
 input: hello world, how are you today?output: Hello World, How Are You Today?
```

The `format_string`  function accepts a string and optional formatter object, and then  applies the formatter to that string. If no formatter is supplied, it  creates a formatter  of its own as a local class and instantiates it. Since it is created  inside the scope of the function, this class cannot be accessed from  anywhere outside of that function. Similarly, functions can be defined  inside other functions as well; in general, any Python statement can be  executed at any time.

These inner classes and functions are  occasionally useful for one-off items that don't require or deserve  their own scope at the module level, or only make sense inside a single  method. However, it is not common to see Python code that frequently  uses this technique.