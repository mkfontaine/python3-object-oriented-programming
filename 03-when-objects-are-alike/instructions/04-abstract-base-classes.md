While duck typing is useful, it is not always easy to tell in advance if a class is going to fulfill the protocol you require. Therefore, Python introduced the idea of **abstract base classes** (**ABC**s).  Abstract base classes define a set of methods and properties that a  class must implement in order to be considered a duck-type instance of  that class. The class can extend the abstract base class itself in order  to be used as an instance of that class, but it must supply all the  appropriate methods.

In practice, it's rarely necessary to create  new abstract base classes, but we may find occasions to implement  instances of existing ABCs. We'll cover implementing ABCs first, and  then briefly see how to create your own, should you ever need to.

# Using an Abstract Base Class

Most of the abstract base classes that exist in the Python standard library live in the `collections` module. One of the simplest ones is the `Container` class. Let's inspect it in the Python interpreter to see what methods this class requires:

```python
>>> from collections import Container 
>>> Container.__abstractmethods__ 
frozenset(['__contains__']) 
```

So, the `Container` class has exactly one abstract method that needs to be implemented, `__contains__`. You can issue `help(Container.__contains__)` to see what the function signature should look like:

```python
Help on method __contains__ in module _abcoll:
 __contains__(self, x) unbound _abcoll.Container method
```

We can see that `__contains__`  needs to take a single argument. Unfortunately, the help file doesn't  tell us much about what that argument should be, but it's pretty obvious  from the name of the ABC and the single method it implements that this  argument is the value the user is checking to see whether the container  holds.

This method is implemented by `list`, `str`, and `dict` to indicate whether or not a given value is **in** that  data structure. However, we can also define a silly container that  tells us whether a given value is in the set of odd integers:

```python
class OddContainer: 
    def __contains__(self, x): 
        if not isinstance(x, int) or not x % 2: 
            return False 
        return True 
```

Here's the interesting part: we can instantiate an `OddContainer` object and determine that, even though we did not extend `Container`, the class is a `Container` object:

```python
>>> from collections import Container 
>>> odd_container = OddContainer() 
>>> isinstance(odd_container, Container) 
True 
>>> issubclass(OddContainer, Container) 
True
```

And that is why duck typing is way more awesome  than classical polymorphism. We can create is a relationships without  the overhead of writing the code to set up inheritance (or worse,  multiple inheritance).

One cool thing about the `Container` ABC is that any class that implements it gets to use the `in` keyword for free. In fact, `in` is just syntax sugar that delegates to the `__contains__` method. Any class that has a `__contains__` method is a `Container` and can therefore be queried by the `in` keyword, for example:

```python
>>> 1 in odd_container 
True 
>>> 2 in odd_container 
False 
>>> 3 in odd_container 
True 
>>> "a string" in odd_container 
False
```

# Creating an Abstract Base Class

As we saw earlier, it's not necessary  to have an abstract base class to enable duck typing. However, imagine  we were creating a media player with third-party plugins. It is  advisable to create an abstract base class in this case to document what  API the third-party plugins should provide (documentation is one of the  stronger use cases for ABCs). The `abc`  module provides the tools you need to do this, but I'll warn you in  advance, this utilizes some of Python's most arcane concepts, as  demonstrated in the following block of code::

```python
import abc 
 
class MediaLoader(metaclass=abc.ABCMeta):
    @abc.abstractmethod
    def play(self):
        pass

    @abc.abstractproperty
    def ext(self):
        pass

    @classmethod
    def __subclasshook__(cls, C):
        if cls is MediaLoader:
            attrs = set(dir(C))
            if set(cls.__abstractmethods__) <= attrs:
                return True

        return NotImplemented
```

This is a complicated  example that includes several Python features that won't be explained  until later in this book. It is included here for completeness, but you  do not need to understand all of it to get the gist of how to create  your own ABC.

The first weird thing is the `metaclass`  keyword argument that is passed into the class where you would normally  see the list of parent classes. This is a seldom-used construct from  the mystic art of metaclass programming. We won't be covering metaclasses in this book, so all you need to know is that by assigning the `ABCMeta` metaclass, you are giving your class superhero (or at least superclass) abilities.

Next, we see the `@abc.abstractmethod` and `@abc.abstractproperty` constructs. These are Python decorators. We'll discuss those in **Python Design Patterns I**.  For now, just know that by marking a method or property as being  abstract, you are stating that any subclass of this class must implement  that method or supply that property in order to be considered a proper  member of the class.

See what happens if you implement subclasses that do, or don't, supply those properties:

```python
>>> class Wav(MediaLoader): 
...     pass 
... 
>>> x = Wav() 
Traceback (most recent call last): 
  File "<stdin>", line 1, in <module> 
TypeError: Can't instantiate abstract class Wav with abstract methods ext, play 
>>> class Ogg(MediaLoader): 
...     ext = '.ogg' 
...     def play(self): 
...         pass 
... 
>>> o = Ogg()
```

Since the `Wav`  class fails to implement the abstract attributes, it is not possible to  instantiate that class. The class is still a legal abstract class, but  you'd have to subclass it to actually do anything. The `Ogg` class supplies both attributes, so it instantiates cleanly.

Going back to the `MediaLoader` ABC, let's dissect that `__subclasshook__`  method. It is basically saying that any class that supplies concrete  implementations of all the abstract attributes of this ABC should be  considered a subclass of `MediaLoader`, even if it doesn't actually inherit from the `MediaLoader` class.

More  common object-oriented languages have a clear separation between the  interface and the implementation of a class. For example, some languages  provide an explicit `interface` keyword that  allows us to define the methods that a class must have without any  implementation. In such an environment, an abstract class is one that  provides both an interface and a concrete implementation of some, but not all, methods. Any class can explicitly state that it implements a given interface.

Python's ABCs help to supply the functionality of interfaces without compromising on the benefits of duck typing.

# Demystifying the Magic

You can copy and paste the subclass code without understanding  it if you want to make abstract classes that fulfill this particular  contract. We'll cover most of the unusual syntaxes in the course, but  let's go over it line by line to get an overview:

```python
    @classmethod 
```

This  decorator marks the method as a class method. It essentially says that  the method can be called on a class instead of an instantiated object:

```python
    def __subclasshook__(cls, C): 
```

This defines the `__subclasshook__` class method. This special method is called by the Python interpreter to answer the question: Is the class `C` a subclass of this class?

```python
        if cls is MediaLoader: 
```

We  check to see whether the method was called specifically on this class,  rather than, say, a subclass of this class. This prevents, for example,  the `Wav` class from being thought of as a parent class of the `Ogg` class:

```python
            attrs = set(dir(C)) 
```

All  this line does is get the set of methods and properties that the class  has, including any parent classes in its class hierarchy:

```python
            if set(cls.__abstractmethods__) <= attrs: 
```

This line uses set notation  to see whether the set of abstract methods in this class has been  supplied in the candidate class. We'll cover sets in detail in the **Python Data Structures**. Note that it doesn't check to see whether the methods have been  implemented; just if they are there. Thus, it's possible for a class to  be a subclass and yet still be an abstract class itself.

```python
                return True 
```

If all the abstract methods have been supplied, then the candidate class is a subclass of this class and we return `True`. The method can legally return one of the three values: `True`, `False`, or `NotImplemented`. `True` and `False` indicate that the class is, or isn't, definitively a subclass of this class:

```python
return NotImplemented 
```

If any of the conditionals have not been met (that is, the class is not `MediaLoader` or not all abstract methods have been supplied), then return `NotImplemented`.  This tells the Python machinery to use the default mechanism (does the  candidate class explicitly extend this class?) for subclass detection.

In short, we can now define the `Ogg` class as a subclass of the `MediaLoader` class without actually extending the `MediaLoader` class:

```python
>>> class Ogg(): ... ext = '.ogg' ... def play(self): ... print("this will play an ogg file") ... >>> issubclass(Ogg, MediaLoader) True >>> isinstance(Ogg(), MediaLoader) True
```