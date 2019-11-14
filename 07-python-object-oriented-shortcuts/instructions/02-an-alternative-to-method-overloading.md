One prominent feature of many object-oriented programming languages is a tool called **method overloading**.  Method overloading simply refers to having multiple methods with the  same name that accept different sets of arguments. In statically typed  languages, this is useful if we want to have a method that accepts  either an integer or a string, for example. In non-object-oriented  languages, we might need two functions, called `add_s` and `add_i`, to accommodate such situations. In statically typed object-oriented languages, we'd need two methods, both called `add`, one that accepts strings, and one that accepts integers.

In  Python, we've already seen that we only need one method, which accepts  any type of object. It may have to do some testing on the object type  (for example, if it is a string, convert it to an integer), but only one  method is required.

However, method overloading is also useful  when we want a method with the same name to accept different numbers or  sets of arguments. For example, an email message method might come in  two versions, one of which accepts an argument for the **from** email address. The other method might look up a default *from* email  address instead. Python doesn't permit multiple methods with the same  name, but it does provide a different, equally flexible, interface.

We've  seen some of the possible ways to send arguments to methods and  functions in previous examples, but now we'll cover all the details. The  simplest function accepts no arguments. We probably don't need an  example, but here's one for completeness:

```python
def no_args(): 
    pass 
```

And here's how it's called:

```python
no_args() 
```

A function that does accept arguments will provide the names of those arguments in a comma-separated list. Only the name of each argument needs to be supplied.

When  calling the function, these positional arguments must be specified in  order, and none can be missed or skipped. This is the most common way in  which we've specified arguments in our previous examples:

```python
def mandatory_args(x, y, z): 
    pass 
```

To call it, type the following::

```python
mandatory_args("a string", a_variable, 5) 
```

Any  type of object can be passed as an argument: an object, a container, a  primitive, even functions and classes. The preceding call shows a  hardcoded string, an unknown variable, and an integer passed into the  function.

# Default Arguments

If we want to make an argument optional, rather than creating  a second method with a different set of arguments, we can specify a  default value in a single method, using an equals sign. If the calling  code does not supply this argument, it will be assigned a default value.  However, the calling code can still choose to override the default by  passing in a different value. Often, a default value of `None`, or an empty string or list, is suitable.

Here's a function definition with default arguments:

```python
def default_arguments(x, y, z, a="Some String", b=False): 
    pass 
```

The first three arguments are still  mandatory and must be passed by the calling code. The last two  parameters have default arguments supplied.

There are several ways  we can call this function. We can supply all arguments in order, as  though all the arguments were positional arguments, as can be seen in  the following:

```python
default_arguments("a string", variable, 8, "", True) 
```

Alternatively,  we can supply just the mandatory arguments in order, leaving the  keyword arguments to be assigned their default values:

```python
default_arguments("a longer string", some_variable, 14) 
```

We  can also use the equals sign syntax when calling a function to provide  values in a different order, or to skip default values that we aren't  interested in. For example, we can skip the first keyword arguments and  supply the second one:

```python
default_arguments("a string", variable, 14, b=True) 
```

Surprisingly, we can even use the equals sign syntax to mix up the order of positional arguments, so long as all of them are supplied:

```python
>>> default_arguments(y=1,z=2,x=3,a="hi")3 1 2 hi False
```

You may occasionally find it useful to make a **keyword-only** argument, that is, an argument that must be supplied as a keyword argument. You can do that by placing a `*` before the keyword-only arguments:

```python
def kw_only(x, y='defaultkw', *, a, b='only'):
    print(x, y, a, b)
```

This function has one positional argument, `x`, and three keyword arguments, `y`, `a`, and `b`. `x` and `y` are both mandatory, but `a` can only be passed as a keyword argument. `y` and `b` are both optional with default values, but if `b` is supplied, it can only be a keyword argument.

This function fails if you don't pass `a`:

```python
>>> kw_only('x')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: kw_only() missing 1 required keyword-only argument: 'a'
```

It also fails if you pass `a` as a positional argument:

```python
>>> kw_only('x', 'y', 'a')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: kw_only() takes from 1 to 2 positional arguments but 3 were given
```

But you can pass `a` and `b` as keyword arguments:

```python
>>> kw_only('x', a='a', b='b')
x defaultkw a b
```

With  so many options, it may seem hard to pick one, but if you think of the  positional arguments as an ordered list, and keyword arguments as sort  of like a dictionary, you'll find that the correct layout tends to fall  into place. If you need to require the caller to specify an argument,  make it mandatory; if you have a sensible default, then make it a  keyword argument. Choosing how to call the method  normally takes care of itself, depending on which values need to be  supplied, and which can be left at their defaults.  Keyword-only  arguments are relatively rare, but when the use case comes up, they can  make for a more elegant API.

One thing to take note of with  keyword arguments is that anything we provide as a default argument is  evaluated when the function is first interpreted, not when it is called.  This means we can't have dynamically generated default values. For  example, the following code won't behave quite as expected:

```python
number = 5 
def funky_function(number=number): 
    print(number) 
 
number=6 
funky_function(8) 
funky_function() 
print(number) 
```

If we run this code, it outputs the number `8` first, but then it outputs the number `5` for the call with no arguments. We had set the variable to the number `6`, as evidenced by the last line of output, but when the function is called, the number `5` is printed; the default value was calculated when the function was defined, not when it was called.

This  is tricky with empty containers such as lists, sets, and dictionaries.  For example, it is common to ask calling code to supply a list that our  function is going to manipulate, but the list is optional. We'd like to make  an empty list as a default argument. We can't do this; it will create  only one list, when the code is first constructed, demonstrated as  follows:

```python
//DON'T DO THIS
>>> def hello(b=[]):...     b.append('a')...     print(b)...>>> hello()['a']>>> hello()['a', 'a']
```

Whoops, that's not quite what we expected! The usual way to get around this is to make the default value `None`, and then use the `iargument = argument if argument else []`idiom inside the method. Pay close attention!

# Variable Argument Lists

Default values alone do not allow us all the flexible  benefits of method overloading. One thing that makes Python really  slick is the ability to write methods that accept an arbitrary number of  positional or keyword arguments without explicitly naming them. We can  also pass arbitrary lists and dictionaries into such functions.

For example, a function to accept a link or list of links and download the web pages could use such variadic arguments, or **varargs**. Instead of accepting  a single value that is expected to be a list of links, we can accept an  arbitrary number of arguments, where each argument is a different link.  We do this by specifying the `*` operator in the function definition, as follows:

```python
def get_pages(*links): 
    for link in links: 
        #download the link with urllib 
        print(link) 
```

The `*links` parameter says, **I'll accept any number of arguments and put them all in a list named**`links`.  If we supply only one argument, it'll be a list with one element; if we  supply no arguments, it'll be an empty list. Thus, all these function  calls are valid:

```python
get_pages() 
get_pages('http://www.archlinux.org') 
get_pages('http://www.archlinux.org', 
        'http://ccphillips.net/') 
```

We can also accept arbitrary keyword arguments. These arrive in the function as a dictionary. They are specified with two asterisks (as in `**kwargs`) in the function declaration. This tool is commonly used in configuration setups. The following class allows us to specify a set of options with default values:

```python
class Options: 
    default_options = { 
            'port': 21, 
            'host': 'localhost', 
            'username': None, 
            'password': None, 
            'debug': False, 
            } 
    def __init__(self, **kwargs): 
        self.options = dict(Options.default_options) 
        self.options.update(kwargs) 
 
    def __getitem__(self, key): 
        return self.options[key] 
```

All the interesting stuff in this class happens in the `__init__` method. We have a dictionary of default options and values at the class level. The first thing the `__init__`  method does is make a copy of this dictionary. We do that instead of  modifying the dictionary directly, in case we instantiate two separate  sets of options. (Remember, class-level variables are shared between  instances of the class.) Then, `__init__` uses the `update` method on the new dictionary to change any non-default values to those supplied as keyword arguments. The `__getitem__` method simply allows us to use the new class using indexing syntax. Here's a session demonstrating the class in action:

```python
>>> options = Options(username="dusty", password="drowssap",        debug=True)>>> options['debug']True>>> options['port']21>>> options['username']'dusty'
```

We're able to access our `options` instance using dictionary indexing syntax, and the dictionary includes both default values and the ones we set using keyword arguments.

The keyword argument syntax can be dangerous, as it may break the **explicit is better than implicit** rule. In the preceding example, it's possible to pass arbitrary keyword arguments to the `Options`  initializer to represent options that don't exist in the default  dictionary. This may not be a bad thing, depending on the purpose of the  class, but it makes it hard for someone using the class to discover  what valid options are available. It also makes it easy to enter a  confusing typo (**Debug** instead of **debug**, for example) that adds two options where only one should have existed.

Keyword  arguments are also very useful when we need to accept arbitrary  arguments to pass to a second function, but we don't know what those  arguments will be. We saw this in action in **When Objects Are Alike**  when we were building support for multiple inheritance. We can, of  course, combine the variable argument and variable keyword argument  syntax in one function call, and we can use normal positional and  default arguments as well. The following example is somewhat contrived,  but demonstrates the four types in action:

```python
import shutil
import os.path


def augmented_move(
    target_folder, *filenames, verbose=False, **specific
):
    """Move all filenames into the target_folder, allowing
    specific treatment of certain files."""

    def print_verbose(message, filename):
        """print the message only if verbose is enabled"""
        if verbose:
            print(message.format(filename))

    for filename in filenames:
        target_path = os.path.join(target_folder, filename)
        if filename in specific:
            if specific[filename] == "ignore":
                print_verbose("Ignoring {0}", filename)
            elif specific[filename] == "copy":
                print_verbose("Copying {0}", filename)
                shutil.copyfile(filename, target_path)
        else:
            print_verbose("Moving {0}", filename)
            shutil.move(filename, target_path)
```

This  example processes an arbitrary list of files. The first argument is a  target folder, and the default behavior is to move all remaining  non-keyword argument files into that folder. Then there is a keyword-only argument, `verbose`,  which tells us whether to print information on each file processed.  Finally, we can supply a dictionary containing actions to perform on  specific filenames; the default behavior is to move the file, but if a  valid string action has been specified in the keyword arguments, it can  be ignored or copied instead. Notice the ordering of the parameters in  the function; first, the positional argument is specified, then the `*filenames` list, then any specific keyword-only arguments, and finally, a `**specific` dictionary to hold remaining keyword arguments.

We create an inner helper function, `print_verbose`, which will print messages only if the `verbose` key has been set. This function keeps code readable by encapsulating this functionality in a single location.

In common cases, assuming the files in question exist, this function could be called as follows:

```python
>>> augmented_move("move_here", "one", "two")
```

This command would move the files `one` and `two` into the `move_here`  directory, assuming they exist (there's no error checking or exception  handling in the function, so it would fail spectacularly if the files or  target directory didn't exist). The move would occur without any  output, since `verbose` is `False` by default.

If we want to see the output, we can call it with the help of the following command:

```python
>>> augmented_move("move_here", "three", verbose=True)Moving three
```

This moves one file named `three`, and tells us what it's doing. Notice that it is impossible to specify `verbose`  as a positional argument in this example; we must pass a keyword  argument. Otherwise, Python would think it was another filename in the `*filenames` list.

If  we want to copy or ignore some of the files in the list, instead of  moving them, we can pass additional keyword arguments, as follows:

```python
>>> augmented_move("move_here", "four", "five", "six",        four="copy", five="ignore")
```

This will move the sixth file and copy the fourth, but won't display any output, since we didn't specify `verbose`. Of course, we can do that too, and keyword arguments can be supplied in any order, demonstrated as follows:

```python
>>> augmented_move("move_here", "seven", "eight", "nine",        seven="copy", verbose=True, eight="ignore")Copying sevenIgnoring eightMoving nine
```

# Unpacking Arguments

There's one more nifty trick involving variable  arguments and keyword arguments. We've used it in some of our previous  examples, but it's never too late for an explanation. Given a list or  dictionary of values, we can pass those values into a function as if  they were normal positional or keyword arguments. Have a look at this  code:

```python
def show_args(arg1, arg2, arg3="THREE"): 
    print(arg1, arg2, arg3) 
 
some_args = range(3) 
more_args = { 
        "arg1": "ONE", 
        "arg2": "TWO"} 
 
print("Unpacking a sequence:", end=" ") 
 
show_args(*some_args) 
print("Unpacking a dict:", end=" ") 
 
show_args(**more_args)
```

Here's what it looks like when we run it:

```python
Unpacking a sequence: 0 1 2Unpacking a dict: ONE TWO THREE
```

The  function accepts three arguments, one of which has a default value. But  when we have a list of three arguments, we can use the `*` operator inside a function call to unpack it into the three arguments. If we have a dictionary of arguments, we can use the `**` syntax to unpack it as a collection of keyword arguments.

This  is most often useful when mapping information that has been collected  from user input or from an outside source (for example, an internet page  or a text file) to a function or method call.

Remember our  earlier example that used headers and lines in a text file to create a  list of dictionaries with contact information? Instead of just adding  the dictionaries to a list, we could use keyword unpacking to pass the arguments to the `__init__` method on a specially built `Contact` object that accepts the same set of arguments. See if you can adapt the example to make this work.

This unpacking syntax can be used in some areas outside of function calls, too. The `Options` class earlier had an `__init__` method that looked like this:

```python
    def __init__(self, **kwargs):
        self.options = dict(Options.default_options)
        self.options.update(kwargs)
```

An even more succinct way to do this would be to unpack the two dictionaries like this:

```python
    def __init__(self, **kwargs):
        self.options = {**Options.default_options, **kwargs}
```

Because  the dictionaries are unpacked in order from left to right, the  resulting dictionary will contain all the default options, with any of  the kwarg options replacing some of the keys. Here's an example:

```python
>>> x = {'a': 1, 'b': 2}
>>> y = {'b': 11, 'c': 3}
>>> z = {**x, **y}
>>> z
{'a': 1, 'b': 11, 'c': 3}
```

