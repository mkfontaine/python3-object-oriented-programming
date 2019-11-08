We don't have to write much Python code to realize that Python is a very **clean** language.  When we want to do something, we can just do it, without having to set  up a bunch of prerequisite code. The ubiquitous **hello world** in Python, as you've likely seen, is only one line.

Similarly, the simplest class in Python 3 looks like this:

```python
class MyFirstClass: 
    pass 
```

There's our first object-oriented program! The class definition starts with the `class` keyword. This is followed by a name (of our choice) identifying the class, and is terminated with a colon.

info> The  class name must follow standard Python variable naming rules (it must  start with a letter or underscore, and can only be comprised of letters,  underscores, or numbers). In addition, the Python style guide (search  the web for *PEP 8*) recommends that classes should be named using **CapWords** notation (start with a capital letter; any subsequent words should also start with a capital).

The class definition line is followed  by the class contents, indented. As with other Python constructs,  indentation is used to delimit the classes, rather than braces,  keywords, or brackets, as many other languages use. Also in line with  the style guide, use four spaces for indentation unless you have a  compelling reason not to (such as fitting in with somebody else's code  that uses tabs for indents).

Since our first class doesn't actually add any data or behaviors, we simply use the `pass` keyword on the second line to indicate that no further action needs to be taken.

We  might think there isn't much we can do with this most basic class, but  it does allow us to instantiate objects of that class. We can load the  class into the Python 3 interpreter, so we can interactively play with  it. To do this, save the class definition mentioned earlier in a file  named `first_class.py` and then run the `python -i first_class.py` command. The `-i` argument tells Python to **run the code and then drop to the interactive interpreter**. The following interpreter session demonstrates a basic interaction with this class:

```python
>>> a = MyFirstClass()>>> b = MyFirstClass()>>> print(a)<__main__.MyFirstClass object at 0xb7b7faec>>>> print(b)<__main__.MyFirstClass object at 0xb7b7fbac>>>>
```

This code instantiates two objects from the new class, named `a` and `b`.  Creating an instance of a class is a simple matter of typing the class  name, followed by a pair of parentheses. It looks much like a normal  function call, but Python knows we're *calling* a  class and not a function, so it understands that its job is to create a  new object. When printed, the two objects tell us which class they are  and what memory address they live at. Memory addresses aren't used much  in Python code, but here, they demonstrate that there are two distinct objects involved.

# Adding Attributes

Now, we have a basic class, but it's fairly  useless. It doesn't contain any data, and it doesn't do anything. What  do we have to do to assign an attribute to a given object?

In  fact, we don't have to do anything special in the class definition. We  can set arbitrary attributes on an instantiated object using dot  notation:

```python
class Point: 
    pass 
 
p1 = Point() 
p2 = Point() 
 
p1.x = 5 
p1.y = 4 
 
p2.x = 3 
p2.y = 6 
 
print(p1.x, p1.y) 
print(p2.x, p2.y) 
```

If we run this code, the two `print` statements at the end tell us the new attribute values on the two objects:

```reStructuredText
5 43 6
```

This code creates an empty `Point` class with no data or behaviors. Then, it creates two instances of that class and assigns each of those instances `x` and `y` coordinates to identify a point in two dimensions. All we need to do to assign a value to an attribute on an object is use the `<object>.<attribute> = <value>` syntax. This is sometimes referred to as **dot notation**. You have likely encountered this same notation before when reading attributes on objects provided by the standard library  or a third-party library. The value can be anything: a Python  primitive, a built-in data type, or another object. It can even be a  function or another class!

# Making It Do Something

Now, having objects with attributes  is great, but object-oriented programming is really about the  interaction between objects. We're interested in invoking actions that  cause things to happen to those attributes. We have data; now it's time  to add behaviors to our classes.

Let's model a couple of actions on our `Point` class. We can start with a **method** called `reset`, which moves the point to the origin (the origin is the place where `x` and `y` are both zero). This is a good introductory action because it doesn't require any parameters:

```python
class Point: 
    def reset(self): 
        self.x = 0 
        self.y = 0 
 
p = Point() 
p.reset() 
print(p.x, p.y) 
```

This `print` statement shows us the two zeros on the attributes:

```reStructuredText
0 0
```

In Python, a method is formatted identically to a function. It starts with the `def` keyword ,  followed by a space, and the name of the method. This is followed by a  set of parentheses containing the parameter list (we'll discuss that `self`  parameter in just a moment), and terminated with a colon. The next line  is indented to contain the statements inside the method. These  statements can be arbitrary Python code operating on the object itself and any parameters passed in, as the method sees fit.

## Talking to Yourself

The one difference, syntactically, between methods and normal functions is that all methods have one required argument. This argument is conventionally named `self`;  I've never seen a Python programmer use any other name for this  variable (convention is a very powerful thing). There's nothing stopping  you, however, from calling it `this` or even `Martha`.

The `self`  argument to a method is a reference to the object that the method is  being invoked on. We can access attributes and methods of that object as  if it were any another object. This is exactly what we do inside the `reset` method when we set the `x` and `y` attributes of the `self` object.

info> Pay attention to the difference between a **class** and an **object** in this discussion. We can think of the **method** as a function attached to a class. The **self**  parameter is a specific instance of that class. When you call the  method on two different objects, you are calling the same method twice,  but passing two different **objects** as the **self** parameter.

Notice that when we call the `p.reset()` method, we do not have to pass the `self` argument into it. Python automatically takes care of this part for us. It knows we're calling a method on the `p` object, so it automatically passes that object to the method.

However,  the method really is just a function that happens to be on a class.  Instead of calling the method on the object, we could invoke the  function on the class, explicitly passing our object as the `self` argument:

```python
>>> p = Point() 
>>> Point.reset(p) 
>>> print(p.x, p.y)
```

The output is the same as in the previous example because, internally, the exact same process has occurred.

What happens if we forget to include the `self` argument in our class definition? Python will bail with an error message, as follows:

```python
>>> class Point:
... def reset():
... pass
...
>>> p = Point()
>>> p.reset()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: reset() takes 0 positional arguments but 1 was given
```

The error message is not as clear as it could be (Hey, silly, you forgot the `self`  argument would be more informative). Just remember that when you see an  error message that indicates missing arguments, the first thing to  check is whether you forgot `self` in the method definition.

## More Arguments

So, how do we pass multiple arguments  to a method? Let's add a new method that allows us to move a point to  an arbitrary position, not just to the origin. We can also include one  that accepts another `Point` object as input and returns the distance between them:

```python
import math


class Point:
    def move(self, x, y):
        self.x = x
        self.y = y

    def reset(self):
        self.move(0, 0)

 def calculate_distance(self, other_point):
        return math.sqrt(
            (self.x - other_point.x) ** 2
            + (self.y - other_point.y) ** 2
        )


# how to use it:
point1 = Point()
point2 = Point()

point1.reset()
point2.move(5, 0)
print(point2.calculate_distance(point1))
assert point2.calculate_distance(point1) == point1.calculate_distance(
    point2
)
point1.move(3, 4)
print(point1.calculate_distance(point2))
print(point1.calculate_distance(point1))
```

The `print` statements at the end give us the following output:

```reStructuredText
5.04.472135954999580.0
```

A lot has happened here. The class now has three methods. The `move` method accepts two arguments, `x` and `y`, and sets the values on the `self` object, much like the old `reset` method from the previous example. The old `reset` method now calls `move`, since a reset is just a move to a specific known location.

The `calculate_distance`  method uses the not-too-complex Pythagorean theorem to calculate the  distance between two points. I hope you understand the math (`**2` means squared, and `math.sqrt` calculates a square root), but it's not a requirement for our current focus, learning how to write methods.

The  sample code at the end of the preceding example shows how to call a  method with arguments: simply include the arguments inside the  parentheses, and use the same dot notation to access the method. I just  picked some random positions to test the methods. The test code calls  each method and prints the results on the console. The `assert` function is a simple test tool; the program will bail if the statement after `assert` evaluates to `False` (or zero, empty, or `None`). In this case, we use it to ensure that the distance is the same regardless of which point called the other point's `calculate_distance` method.

# Initializing the object

If we don't explicitly set the `x` and `y` positions on our `Point` object, either using `move` or by accessing them directly, we have a broken point with no real position. What will happen when we try to access it?

Well, let's just try it and see. **Try it and see** is  an extremely useful tool for Python study. Open up your interactive  interpreter and type away. The following interactive session shows what  happens if we try to access a missing attribute. If you saved the  previous example as a file or are using the examples distributed with  the book, you can load it into the Python interpreter with the `python -i more_arguments.py` command:

```python
>>> point = Point()>>> point.x = 5>>> print(point.x)5>>> print(point.y)Traceback (most recent call last):  File "<stdin>", line 1, in <module>AttributeError: 'Point' object has no attribute 'y'
```

Well, at least it threw a useful exception. We'll cover exceptions in detail in **Expecting the Unexpected**. You've probably seen them before (especially the ubiquitous **`SyntaxError`**, which means you typed something incorrectly!). At this point, simply be aware that it means something went wrong.

The output is useful for debugging. In the interactive interpreter, it tells us the error occurred at **`line 1`**,  which is only partially true (in an interactive session, only one line  is executed at a time). If we were running a script in a file, it would  tell us the exact line number, making it easy to find the offending  code. In addition, it tells us that the error is an `AttributeError`, and gives a helpful message telling us what that error means.

We  can catch and recover from this error, but in this case, it feels like  we should have specified some sort of default value. Perhaps every new  object should be `reset()` by default, or maybe it would be nice if we could force the user to tell us what those positions should be when they create the object.

Most object-oriented programming languages have the concept of a **constructor**,  a special method that creates and initializes the object when it is  created. Python is a little different; it has a constructor **and** an initializer. The constructor function is rarely used, unless you're  doing something very exotic. So, we'll start our discussion with the  much more common initialization method.

The Python initialization method is the same as any other method, except it has a special name, `__init__`.  The leading and trailing double underscores mean this is a special  method that the Python interpreter will treat as a special case.

info> Never  name a method of your own with leading and trailing double underscores.  It may mean nothing to Python today, but there's always the possibility  that the designers of Python will add a function that has a special  purpose with that name in the future, and when they do, your code will  break.

Let's add an initialization function on our `Point` class that requires the user to supply `x` and `y` coordinates when the `Point` object is instantiated:

```python
class Point: 
    def __init__(self, x, y): 
        self.move(x, y) 
 
    def move(self, x, y): 
        self.x = x 
        self.y = y 
 
    def reset(self): 
        self.move(0, 0) 
 
# Constructing a Point 
point = Point(3, 5) 
print(point.x, point.y) 
```

Now, our point can never go without a `y` coordinate! If we try to construct a point without including the proper initialization parameters, it will fail with a `not enough arguments` error similar to the one we received earlier when we forgot the `self` argument.

If  we don't want to make the two arguments required, we can use the same  syntax Python functions use to provide default arguments. The keyword  argument syntax appends an equals sign after each variable name. If the  calling object does not provide this argument, then the default argument  is used instead. The variables will still be available to the function,  but they will have the values specified in the argument list. Here's an  example:

```python
class Point: 
    def __init__(self, x=0, y=0): 
        self.move(x, y) 
```

Most of the time, we put our initialization statements in an `__init__`  function. But as mentioned earlier, Python has a constructor in  addition to its initialization function. You may never need to use the  other Python constructor (in well over a decade of professional Python  coding, I can only think of two cases where I've used it, and in one of  them, I probably shouldn't have!), but it helps to know it exists, so  we'll cover it briefly.

The constructor function is called `__new__` as opposed to `__init__`, and accepts exactly one argument; the **class** that is being constructed (it is called **before** the object is constructed, so there is no `self`  argument). It also has to return the newly created object. This has  interesting possibilities when it comes to the complicated art of  metaprogramming, but is not very useful in day-to-day Python. In  practice, you will rarely, if ever, need to use `__new__`. The `__init__` method will almost always be sufficient.

# Explaining Yourself

Python is an extremely  easy-to-read programming language; some might say it is  self-documenting. However, when carrying out object-oriented  programming, it is important to write API documentation that clearly  summarizes what each object and method does. Keeping documentation up to  date is difficult; the best way to do it is to write it right into our  code.

Python supports this through the use of **docstrings**.  Each class, function, or method header can have a standard Python  string as the first line following the definition (the line that ends in  a colon). This line should be indented the same as the code that  follows it.

Docstrings are simply Python strings enclosed with apostrophes (`'`) or quotation marks (`"`)  characters. Often, docstrings are quite long and span multiple lines  (the style guide suggests that the line length should not exceed 80  characters), which can be formatted as multi-line strings, enclosed in  matching triple apostrophe (`'''`) or triple quote (`"""`) characters.

A  docstring should clearly and concisely summarize the purpose of the  class or method it is describing. It should explain any parameters whose  usage is not immediately obvious, and is also a good place to include  short examples of how to use the API. Any caveats or problems an  unsuspecting user of the API should be aware of should also be noted.

To illustrate the use of docstrings, we will end this section with our completely documented `Point` class:

```python
import math


class Point:
    "Represents a point in two-dimensional geometric coordinates"

    def __init__(self, x=0, y=0):
        """Initialize the position of a new point. The x and y
           coordinates can be specified. If they are not, the
           point defaults to the origin."""
        self.move(x, y)

    def move(self, x, y):
        "Move the point to a new location in 2D space."
        self.x = x
        self.y = y

    def reset(self):
        "Reset the point back to the geometric origin: 0, 0"
        self.move(0, 0)

    def calculate_distance(self, other_point):
        """Calculate the distance from this point to a second
        point passed as a parameter.

        This function uses the Pythagorean Theorem to calculate
        the distance between the two points. The distance is
        returned as a float."""

        return math.sqrt(
            (self.x - other_point.x) ** 2
            + (self.y - other_point.y) ** 2
        )
```

Try typing or loading (remember, it's `python -i point.py`) this file into the interactive interpreter. Then, enter `help(Point)<enter>` at the Python prompt.

You should see nicely formatted documentation for the class, as shown in the following screenshot:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/8cc5f0b8-2531-441b-9cb0-51ced3f2d4e6.png)

