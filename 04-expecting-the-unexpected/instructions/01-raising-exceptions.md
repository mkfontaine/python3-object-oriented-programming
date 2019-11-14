In principle, an exception  is just an object. There are many different exception classes  available, and we can easily define more of our own. The one thing they  all have in common is that they inherit from a built-in class called `BaseException`.  These exception objects become special when they are handled inside the  program's flow of control. When an exception occurs, everything that  was supposed to happen doesn't happen, unless it was supposed to happen  when an exception occurred. Make sense? Don't worry, it will!

The  easiest way to cause an exception to occur is to do something silly.  Chances are you've done this already and seen the exception output. For  example, any time Python encounters a line in your program that it can't  understand, it bails with `SyntaxError`, which is a type of exception. Here's a common one:

```python
>>> print "hello world"  File "<stdin>", line 1    print "hello world"                      ^SyntaxError: invalid syntax
```

This `print` statement was a valid command way back in the Python 2 and earlier days, but in Python 3, because `print`  is a function, we have to enclose the arguments in parentheses. So, if  we type the preceding command into a Python 3 interpreter, we get `SyntaxError`.

In addition to `SyntaxError`, some other common exceptions are shown in the following example:

```python
>>> x = 5 / 0Traceback (most recent call last):  File "<stdin>", line 1, in <module>ZeroDivisionError: int division or modulo by zero>>> lst = [1,2,3]>>> print(lst[3])Traceback (most recent call last):  File "<stdin>", line 1, in <module>IndexError: list index out of range>>> lst + 2Traceback (most recent call last):  File "<stdin>", line 1, in <module>TypeError: can only concatenate list (not "int") to list>>> lst.addTraceback (most recent call last):  File "<stdin>", line 1, in <module>AttributeError: 'list' object has no attribute 'add'>>> d = {'a': 'hello'}>>> d['b']Traceback (most recent call last):  File "<stdin>", line 1, in <module>KeyError: 'b'>>> print(this_is_not_a_var)Traceback (most recent call last):  File "<stdin>", line 1, in <module>NameError: name 'this_is_not_a_var' is not defined
```

Sometimes,  these exceptions are indicators of something wrong in our program (in  which case, we would go to the indicated line number and fix it), but  they also occur in legitimate situations. A `ZeroDivisionError` error doesn't always mean we received an invalid  input. It could also mean we have received a different input. The user  may have entered a zero by mistake, or on purpose, or it may represent a  legitimate value, such as an empty bank account or the age of a newborn  child.

You may have noticed all the preceding built-in exceptions end with the name `Error`. In Python, the words `error` and `Exception`  are used almost interchangeably. Errors are sometimes considered more  dire than exceptions, but they are dealt with in exactly the same way.  Indeed, all the error classes in the preceding example have `Exception` (which extends `BaseException`) as their superclass.

# Raising an Exception

We'll get to responding to such exceptions  in a minute, but first, let's discover what we should do if we're  writing a program that needs to inform the user or a calling function  that the inputs are invalid. We can use the exact same mechanism that  Python uses. Here's a simple class that adds items to a list only if  they are even numbered integers:

```python
class EvenOnly(list): 
    def append(self, integer): 
        if not isinstance(integer, int): 
            raise TypeError("Only integers can be added") 
        if integer % 2: 
            raise ValueError("Only even numbers can be added") 
        super().append(integer) 
```

This class extends the `list` built-in, as we discussed in **Objects in Python**, and overrides the `append` method to check two conditions that ensure the item is an even integer. We first check whether the input is an instance of the `int` type, and then use the modulus operator to ensure it is divisible by two. If either of the two conditions is not met, the `raise` keyword causes an exception to occur. The `raise`  keyword is followed by the object being raised as an exception. In the  preceding example, two objects are constructed from the built-in `TypeError` and `ValueError` classes. The raised object could just as easily be an instance of a new `Exception` class we create ourselves (we'll see how shortly), an exception that was defined elsewhere, or even an `Exception` object that has been previously raised and handled.

If we test this class in the Python interpreter, we can see that it is outputting useful error information when exceptions occur, just as before:

```python
>>> e = EvenOnly()>>> e.append("a string")Traceback (most recent call last):  File "<stdin>", line 1, in <module>  File "even_integers.py", line 7, in add    raise TypeError("Only integers can be added")TypeError: Only integers can be added>>> e.append(3)Traceback (most recent call last):  File "<stdin>", line 1, in <module>  File "even_integers.py", line 9, in add    raise ValueError("Only even numbers can be added")ValueError: Only even numbers can be added>>> e.append(2)
```

info> While  this class is effective for demonstrating exceptions in action, it  isn't very good at its job. It is still possible to get other values  into the list using index notation or slice notation. This can all be  avoided by overriding other appropriate methods, some of which are magic  double-underscore methods.

# The Effects of an Exception

When an exception is raised, it appears to stop program  execution immediately. Any lines that were supposed to run after the  exception is raised are not executed, and unless the exception is dealt  with, the program will exit with an error message. Take a look at this  basic function:

```python
def no_return(): 
    print("I am about to raise an exception") 
    raise Exception("This is always raised") 
    print("This line will never execute") 
    return "I won't be returned" 
```

If we execute this function, we see that the first `print` call is executed and then the exception is raised. The second `print` function call is never executed, nor is the `return` statement:

```python
>>> no_return()I am about to raise an exceptionTraceback (most recent call last):  File "<stdin>", line 1, in <module>  File "exception_quits.py", line 3, in no_return    raise Exception("This is always raised")Exception: This is always raised
```

Furthermore,  if we have a function that calls another function that raises an  exception, nothing is executed in the first function after the point  where the second function was called. Raising an exception stops all  execution right up through the function call stack until it is either handled or forces the interpreter to exit. To demonstrate, let's add a second function that calls the earlier one:

```python
def call_exceptor(): 
    print("call_exceptor starts here...") 
    no_return() 
    print("an exception was raised...") 
    print("...so these lines don't run") 
```

When we call this function, we see that the first `print` statement executes, as well as the first line in the `no_return` function. But once the exception is raised, nothing else executes:

```python
>>> call_exceptor()call_exceptor starts here...I am about to raise an exceptionTraceback (most recent call last):  File "<stdin>", line 1, in <module>  File "method_calls_excepting.py", line 9, in call_exceptor    no_return()  File "method_calls_excepting.py", line 3, in no_return    raise Exception("This is always raised")Exception: This is always raised
```

We'll  soon see that when the interpreter is not actually taking a shortcut  and exiting immediately, we can react to and deal with the exception  inside either method. Indeed, exceptions can be handled at any level  after they are initially raised.

Look at the exception's output (called a traceback) from bottom to top, and notice how both methods are listed. Inside `no_return`, the exception is initially raised. Then, just above that, we see that inside `call_exceptor`, that pesky `no_return` function was called and the exception **bubbled up** to the calling method. From there, it went up one more level to the main interpreter, which, not knowing what else to do with it, gave up and printed a traceback.

# Handling Exceptions

Now let's look at the tail side of the exception  coin. If we encounter an exception situation, how should our code react  to or recover from it? We handle exceptions by wrapping any code that  might throw one (whether it is exception code itself, or a call to any  function or method that may have an exception raised inside it) inside a  `try...except` clause. The most basic syntax looks like this:

```python
try: 
    no_return() 
except: 
    print("I caught an exception") 
print("executed after the exception") 
```

If we run this simple script using our existing `no_return` function—which, as we know very well, always throws an exception—we get this output:

```reStructuredText
I am about to raise an exception 
I caught an exception 
executed after the exception
```

The `no_return`  function happily informs us that it is about to raise an exception, but  we fooled it and caught the exception. Once caught, we were able to  clean up after ourselves (in this case, by outputting that we were  handling the situation), and continue on our way, with no interference  from that offensive function. The remainder of the code in the `no_return` function still went unexecuted, but the code that called the function was able to recover and continue.

info> Note the indentation around `try` and `except`. The `try` clause wraps any code that might throw an exception. The `except` clause is then back on the same indentation level as the `try` line. Any code to handle the exception is indented after the `except` clause. Then normal code resumes at the original indentation level.

The problem with the preceding code is that it will catch any type of exception. What if we were writing some code that could raise both `TypeError` and `ZeroDivisionError`? We might want to catch `ZeroDivisionError`, but let `TypeError` propagate to the console. Can you guess the syntax?

Here's a rather silly function that does just that:

```python
def funny_division(divider):
    try:
        return 100 / divider
    except ZeroDivisionError:
        return "Zero is not a good idea!"


print(funny_division(0))
print(funny_division(50.0))
print(funny_division("hello"))
```

The function is tested with the `print` statements that show it behaving as expected:

```python
Zero is not a good idea!2.0Traceback (most recent call last):  File "catch_specific_exception.py", line 9, in <module>    print(funny_division("hello"))  File "catch_specific_exception.py", line 3, in funny_division    return 100 / dividerTypeError: unsupported operand type(s) for /: 'int' and 'str'.
```

The first line of output shows that if we enter `0`,  we get properly mocked. If we call with a valid number (note that it's  not an integer, but it's still a valid divisor), it operates correctly.  Yet if we enter a string (you were wondering how to get a `TypeError`, weren't you?), it fails with an exception. If we had used an empty `except` clause that didn't specify a `ZeroDivisionError`, it would have accused us of dividing by zero when we sent it a string, which is not a proper behavior at all.

info> The **bare except** syntax is generally frowned upon, even if you really do want to catch all instances of an exception. Use the `except Exception:`  syntax to explicitly catch all exception types. This tell the reader  that you meant to catch exception objects and all subclasses of `Exception`. The bare except syntax is actually the same as using `except BaseException:`,  which actually catches system-level exceptions that are very rare to  intentionally want to catch, as we'll see in the next section. If you  really do want to catch them, explicitly use `except BaseException:` so that anyone who reads your code knows that you didn't just forget to specify what kind of exception you wanted.

We  can even catch two or more different exceptions and handle them with  the same code. Here's an example that raises three different types of  exception. It handles `TypeError` and `ZeroDivisionError` with the same exception handler, but it may also raise a `ValueError` error if you supply the number `13`:

```python
def funny_division2(divider):
    try:
        if divider == 13:
            raise ValueError("13 is an unlucky number")
        return 100 / divider
    except (ZeroDivisionError, TypeError):
        return "Enter a number other than zero"


for val in (0, "hello", 50.0, 13):

    print("Testing {}:".format(val), end=" ")
    print(funny_division2(val))
```

The `for` loop at the bottom loops over several test inputs and prints the results. If you're wondering about that `end` argument in the `print` statement, it just turns the default trailing newline into a space so that it's joined with the output from the next line. Here's a run of the program:

```python
Testing 0: Enter a number other than zeroTesting hello: Enter a number other than zeroTesting 50.0: 2.0Testing 13: Traceback (most recent call last):  File "catch_multiple_exceptions.py", line 11, in <module>    print(funny_division2(val))  File "catch_multiple_exceptions.py", line 4, in funny_division2    raise ValueError("13 is an unlucky number")ValueError: 13 is an unlucky number
```

The number `0` and the string are both caught by the `except` clause, and a suitable error message is printed. The exception from the number `13` is not caught because it is a `ValueError`,  which was not included in the types of exceptions being handled. This  is all well and good, but what if we want to catch different exceptions  and do different things with them? Or maybe we want to do something with  an exception and then allow it to continue to bubble up to the parent  function, as if it had never been caught?

We don't need any new syntax to deal with these cases. It's possible to stack the `except` clauses, and only the first match will be executed. For the second question, the `raise` keyword, with no arguments, will re-raise the last exception if we're already inside an exception handler. Observe the following code:

```python
def funny_division3(divider):
    try:
        if divider == 13:
            raise ValueError("13 is an unlucky number")
        return 100 / divider
    except ZeroDivisionError:
        return "Enter a number other than zero"
    except TypeError:
        return "Enter a numerical value"
    except ValueError:
        print("No, No, not 13!")
        raise
```

The last line re-raises the `ValueError` error, so after outputting `No, No, not 13!`, it will raise the exception again; we'll still get the original stack trace on the console.

If  we stack exception clauses like we did in the preceding example, only  the first matching clause will be run, even if more than one of them  fits. How can more than one clause match? Remember that exceptions are  objects, and can therefore be subclassed. As we'll see in the next  section, most exceptions extend the `Exception` class (which is itself derived from `BaseException`). If we catch `Exception` before we catch `TypeError`, then only the `Exception` handler will be executed, because `TypeError` is an `Exception` by inheritance.

This  can come in handy in cases where we want to handle some exceptions  specifically, and then handle all remaining exceptions as a more general  case. We can simply catch `Exception` after catching all the specific exceptions and handle the general case there.

Often, when we catch an exception, we need a reference to the `Exception`  object itself. This most often happens when we define our own  exceptions with custom arguments, but can also be relevant with standard  exceptions. Most exception  classes accept a set of arguments in their constructor, and we might  want to access those attributes in the exception handler. If we define  our own `Exception` class, we can even call custom methods on it when we catch it. The syntax for capturing an exception as a variable uses the `as` keyword:

```python
try: 
    raise ValueError("This is an argument") 
except ValueError as e: 
    print("The exception arguments were", e.args) 
```

If we run this simple snippet, it prints out the string argument that we passed into `ValueError` upon initialization.

We've  seen several variations on the syntax for handling exceptions, but we  still don't know how to execute code regardless of whether or not an  exception has occurred. We also can't specify code that should be  executed **only** if an exception does **not** occur. Two more keywords, `finally` and `else`,  can provide the missing pieces. Neither one takes any extra arguments.  The following example randomly picks an exception to throw and raises  it. Then some not-so-complicated exception handling code runs that  illustrates the newly introduced syntax:

```python
import random 
some_exceptions = [ValueError, TypeError, IndexError, None] 
 
try: 
    choice = random.choice(some_exceptions) 
    print("raising {}".format(choice)) 
    if choice: 
        raise choice("An error") 
except ValueError: 
    print("Caught a ValueError") 
except TypeError: 
    print("Caught a TypeError") 
except Exception as e: 
    print("Caught some other error: %s" % 
        ( e.__class__.__name__)) 
else: 
    print("This code called if there is no exception") 
finally: 
    print("This cleanup code is always called") 
```

If we run this example—which illustrates almost every conceivable exception handling scenario—a few times, we'll get different output each time, depending on which exception `random` chooses. Here are some example runs:

```bash
$ python finally_and_else.pyraising NoneThis code called if there is no exceptionThis cleanup code is always called$ python finally_and_else.pyraising <class 'TypeError'>Caught a TypeErrorThis cleanup code is always called$ python finally_and_else.pyraising <class 'IndexError'>Caught some other error: IndexErrorThis cleanup code is always called$ python finally_and_else.pyraising <class 'ValueError'>Caught a ValueErrorThis cleanup code is always called
```

Note how the `print` statement in the `finally`  clause is executed no matter what happens. This is extremely useful  when we need to perform certain tasks after our code has finished  running (even if an exception has occurred). Some common examples  include the following:

- Cleaning up an open database connection
- Closing an open file
- Sending a closing handshake over the network

info> The `finally` clause is also very important when we execute a `return` statement from inside a `try` clause. The `finally` handler will still be executed before the value is returned without executing any code following the `try...finally` clause.

Also, pay attention to the output when no exception is raised: both the `else` and the `finally` clauses are executed. The `else`  clause may seem redundant, as the code that should be executed only  when no exception is raised could just be placed after the entire `try...except` block. The difference is that the `else`  block will not be executed if an exception is caught and handled. We'll  see more on this when we discuss using exceptions as flow control  later.

Any of the `except`, `else`, and `finally` clauses can be omitted after a `try` block (although `else` by itself is invalid). If you include more than one, the `except` clauses must come first, then the `else` clause, with the `finally` clause at the end. The order of the `except` clauses normally goes from most specific to most generic.

# The Exception Hierarchy

We've  already seen several of the most common built-in exceptions, and you'll  probably encounter the rest over the course of your regular Python  development. As we noticed earlier, most exceptions are subclasses of  the `Exception` class. But this is not true of all exceptions. `Exception` itself actually inherits from a class called `BaseException`. In fact, all exceptions must extend the `BaseException` class or one of its subclasses.

There are two key built-in the exception classes, `SystemExit` and `KeyboardInterrupt`, that derive directly from `BaseException` instead of `Exception`. The `SystemExit` exception is raised whenever the program exits naturally, typically because we called the `sys.exit` function somewhere in our code (for example, when the user selected an exit menu item, clicked the **Close** button  on a window, or entered a command to shut down a server). The exception  is designed to allow us to clean up code before the program ultimately  exits. However, we generally don't need to handle it explicitly because  cleanup code can happen inside a `finally` clause.

If  we do handle it, we would normally re-raise the exception, since  catching it would stop the program from exiting. There are, of course,  situations where we might want to stop the program exiting; for example,  if there are unsaved changes and we want to prompt the user if they  really want to exit. Usually, if we handle `SystemExit`  at all, it's because we want to do something special with it, or are  anticipating it directly. We especially don't want it to be accidentally  caught in generic clauses that catch all normal exceptions. This is why  it derives directly from `BaseException`.

The `KeyboardInterrupt`  exception is common in command-line programs. It is thrown when the  user explicitly interrupts program execution with an OS-dependent key  combination (normally, ***Ctrl*** + ***C***). This is a standard way for the user to deliberately interrupt a running program, and like `SystemExit`, it should almost always respond by terminating the program. Also, like `SystemExit`, it should handle any cleanup tasks inside the `finally` blocks.

Here is a class diagram that fully illustrates the  hierarchy:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/94709e07-2baf-47ee-b1fc-4d2416f0a66d.png)

When we use the `except:` clause without specifying any type of exception, it will catch all subclasses of `BaseException`;  which is to say, it will catch all exceptions, including the two  special ones. Since we almost always want these to get special  treatment, it is unwise to use the `except:` statement without arguments. If you want to catch all exceptions other than `SystemExit` and `KeyboardInterrupt`, explicitly catch `Exception`. Most Python developers assume that `except:` without a type is an error and will flag it in code review. If you really do want to catch everything, just explicitly use `except BaseException:`.

# Defining Our Own Exceptions

Occasionally,  when we want to raise an exception, we find that none of the built-in  exceptions are suitable. Luckily, it's trivial to define new exceptions of our own. The name of the class is usually designed to communicate what went wrong, and we can provide arbitrary arguments in the initializer to include additional information.

All we have to do is inherit from the `Exception` class. We don't even have to add any content to the class! We can, of course, extend `BaseException` directly, but I have never encountered a use case where this would make sense.

Here's a simple exception we might use in a banking application:

```python
class InvalidWithdrawal(Exception): 
    pass 
 
raise InvalidWithdrawal("You don't have $50 in your account") 
```

The  last line illustrates how to raise the newly defined exception. We are  able to pass an arbitrary number of arguments into the exception. Often a  string message is used, but any object that might be useful in a later  exception handler can be stored. The `Exception.__init__` method is designed to accept any arguments and store them as a tuple in an attribute named `args`. This makes exceptions easier to define without needing to override `__init__`.

Of  course, if we do want to customize the initializer, we are free to do  so. Here's an exception whose initializer accepts the current balance  and the amount the user wanted to withdraw. In addition, it adds a  method to calculate how overdrawn the request was:

```python
class InvalidWithdrawal(Exception): 
    def __init__(self, balance, amount): 
        super().__init__(f"account doesn't have ${amount}") 
        self.amount = amount 
        self.balance = balance 
 
    def overage(self): 
        return self.amount - self.balance 
 
raise InvalidWithdrawal(25, 50)
```

The `raise` statement at the end illustrates how to construct this exception. As you can see, we can do anything with an exception that we would do with other objects.

Here's how we would handle an `InvalidWithdrawal` exception if one was raised:

```python
try: 
    raise InvalidWithdrawal(25, 50) 
except InvalidWithdrawal as e: 
    print("I'm sorry, but your withdrawal is " 
            "more than your balance by " 
            f"${e.overage()}") 
```

Here we see a valid use of the `as` keyword. By convention, most Python coders name the exception `e` or the `ex` variable, although, as usual, you are free to call it  `exception`, or `aunt_sally` if you prefer.

There  are many reasons for defining our own exceptions. It is often useful to  add information to the exception or log it in some way. But the utility  of custom exceptions truly comes to light when creating a framework,  library, or API that is intended for access by other programmers. In  that case, be careful to ensure your code is raising exceptions that  make sense to the client programmer. They should be easy to handle and  clearly describe what went on. The client programmer should easily see  how to fix the error (if it reflects a bug in their code) or handle the  exception (if it's a situation they need to be made aware of).

Exceptions  aren't exceptional. Novice programmers tend to think of exceptions as  only useful for exceptional circumstances. However, the definition of  exceptional circumstances can be vague and subject to interpretation.  Consider the following two functions:

```python
def divide_with_exception(number, divisor): 
    try: 
        print(f"{number} / {divisor} = {number / divisor}") 
    except ZeroDivisionError: 
        print("You can't divide by zero") 
 
def divide_with_if(number, divisor): 
    if divisor == 0: 
        print("You can't divide by zero") 
    else: 
        print(f"{number} / {divisor} = {number / divisor}") 
```

These two functions behave identically. If `divisor` is zero, an error message is printed; otherwise, a message printing the result of division is displayed. We could avoid `ZeroDivisionError` ever being thrown by testing for it with an `if` statement. Similarly, we can avoid `IndexError` by explicitly checking whether or not the parameter is within the confines of the list, and `KeyError` by checking whether the key is in a dictionary.

But we shouldn't do this. For one thing, we might write an `if` statement that checks whether or not the index is lower than the parameters of the list, but forget to check negative values.

info> Remember, Python lists support negative indexing; `-1` refers to the last element in the list.

Eventually, we would discover this and have to find all the places where we were checking code. But if we had simply caught `IndexError` and handled it, our code would just work.

Python programmers tend to follow a model of **ask forgiveness rather than permission**, which is to say, they execute code and then deal with anything that goes wrong. The alternative, to **look before you leap**,  is generally less popular. There are a few reasons for this, but the  main one is that it shouldn't be necessary to burn CPU cycles looking  for an unusual situation that is not going to arise in the normal path  through the code. Therefore, it is wise to use exceptions for  exceptional circumstances, even if those circumstances are only a little  bit exceptional. Taking this argument further, we can actually see that  the exception syntax is also effective for flow control. Like an `if` statement, exceptions can be used for decision making, branching, and message passing.

Imagine  an inventory application for a company that sells widgets and gadgets.  When a customer makes a purchase, the item can either be available, in  which case the item is removed from inventory and the number of items  left is returned, or it might be out of stock. Now, being out of stock  is a perfectly normal thing to happen in an inventory application. It is  certainly not an exceptional circumstance. But what do we return if  it's out of stock? A string saying out of stock? A negative number? In  both cases, the calling method would have to check whether the return  value is a positive integer or something else, to determine if it is out  of stock. That seems a bit messy, especially if we forget to do it  somewhere in our code.

Instead, we can raise `OutOfStock` and use the `try`  statement to direct program flow control. Make sense? In addition, we  want to make sure we don't sell the same item to two different  customers, or sell an item that isn't in stock yet. One way to facilitate this is to lock  each type of item to ensure only one person can update it at a time.  The user must lock the item, manipulate the item (purchase, add stock,  count items left...), and then unlock the item. Here's an incomplete `Inventory` example with docstrings that describes what some of the methods should do:

```python
class Inventory:
    def lock(self, item_type):
        """Select the type of item that is going to
        be manipulated. This method will lock the
        item so nobody else can manipulate the
        inventory until it's returned. This prevents
        selling the same item to two different
        customers."""
        pass

    def unlock(self, item_type):
        """Release the given type so that other
        customers can access it."""
        pass

    def purchase(self, item_type):
        """If the item is not locked, raise an
        exception. If the item_type does not exist,
        raise an exception. If the item is currently
        out of stock, raise an exception. If the item
        is available, subtract one item and return
        the number of items left."""
        pass
```

We could hand this object prototype to  a developer and have them implement the methods to do exactly as they  say while we work on the code that needs to make a purchase. We'll use  Python's robust exception handling to consider different branches, depending on how the purchase was made:

```python
item_type = "widget"
inv = Inventory()
inv.lock(item_type)
try:
    num_left = inv.purchase(item_type)
except InvalidItemType:
    print("Sorry, we don't sell {}".format(item_type))
except OutOfStock:
    print("Sorry, that item is out of stock.")
else:
    print("Purchase complete. There are {num_left} {item_type}s left")
finally:
    inv.unlock(item_type)
```

Pay attention to how all  the possible exception handling clauses are used to ensure the correct  actions happen at the correct time. Even though `OutOfStock`  is not a terribly exceptional circumstance, we are able to use an  exception to handle it suitably. This same code could be written with an  `if...elif...else` structure, but it wouldn't be as easy to read or maintain.

Pay attention to how all the possible exception handling clauses are used to ensure the correct actions happen at the correct time. Even though `OutOfStock` is not a terribly 
exceptional circumstance, we are able to use an exception to handle it suitably. This same code could be written with an `if...elif...else` structure, but it wouldn't be as easy to read or maintain.

We  can also use exceptions to pass messages between different methods. For  example, if we wanted to inform the customer as to what date the item  is expected to be in stock again, we could ensure our `OutOfStock` object requires a `back_in_stock`  parameter when it is constructed. Then, when we handle the exception,  we can check that value and provide additional information to the  customer. The information attached to the object can be easily passed  between two different parts of the program. The exception could even  provide a method that instructs the inventory object to reorder or  backorder an item.

Using exceptions for flow control can make for  some handy program designs. The important thing to take from this  discussion is that exceptions are not a bad thing that we should try to  avoid. Having an exception occur does not mean that you should have  prevented this exceptional circumstance from happening. Rather, it is  just a powerful way to communicate information between two sections of code that may not be directly calling each other.