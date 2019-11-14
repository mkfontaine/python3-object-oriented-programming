Programming languages that overemphasize object-oriented principles  tend to frown on functions that are not methods. In such languages,  you're expected to create an object to sort of wrap the single method  involved. There are numerous situations where we'd like to pass around a  small object that is simply called to perform an action. This is most  frequently done in event-driven programming, such as graphical toolkits  or asynchronous servers; we'll see some design patterns that use it in **Design Patterns I**, and **Design Patterns II**.

In  Python, we don't need to wrap such methods in an object because  functions already are objects! We can set attributes on functions  (though this isn't a common activity), and we can pass them around to be  called at a later date. They even have a few special properties that  can be accessed directly. Here's yet another contrived example:

```python
def my_function():
    print("The Function Was Called")


my_function.description = "A silly function"


def second_function():
    print("The second was called")


second_function.description = "A sillier function."


def another_function(function):
    print("The description:", end=" ")
    print(function.description)
    print("The name:", end=" ")
    print(function.__name__)
    print("The class:", end=" ")
    print(function.__class__)
    print("Now I'll call the function passed in")
    function()


another_function(my_function)
another_function(second_function)
```

If we run this code, we can see that we were able to pass two different functions into our third function, and get different output for each one:

```python
The description: A silly function 
The name: my_function 
The class: <class 'function'> 
Now I'll call the function passed in 
The Function Was Called 
The description: A sillier function. 
The name: second_function 
The class: <class 'function'> 
Now I'll call the function passed in 
The second was called
```

We set an attribute on the function, named `description` (not very good descriptions, admittedly). We were also able to see the function's `__name__`  attribute, and to access its class, demonstrating that the function  really is an object with attributes. Then, we called the function by  using the callable syntax (the parentheses).

The fact that  functions are top-level objects is most often used to pass them around  to be executed at a later date, for example, when a certain condition  has been satisfied. Let's build an event-driven timer that does just  this:

```python
import datetime
import time


class TimedEvent:
    def __init__(self, endtime, callback):
        self.endtime = endtime
        self.callback = callback

    def ready(self):
        return self.endtime <= datetime.datetime.now()


class Timer:
    def __init__(self):
        self.events = []

    def call_after(self, delay, callback):
        end_time = datetime.datetime.now() + datetime.timedelta(
            seconds=delay
        )

        self.events.append(TimedEvent(end_time, callback))

    def run(self):
        while True:
            ready_events = (e for e in self.events if e.ready())
            for event in ready_events:
                event.callback(self)
                self.events.remove(event)
            time.sleep(0.5)
```

In production, this code should definitely have extra documentation using docstrings! The `call_after` method should at least mention that the `delay` parameter is in seconds, and that the `callback` function should accept one argument: the timer doing the calling.

We have two classes here. The `TimedEvent` class is not really meant to be accessed by other classes; all it does is store `endtime` and `callback`. We could even use a `tuple` or `namedtuple`  here, but as it is convenient to give the object a behavior that tells  us whether or not the event is ready to run, we use a class instead.

The `Timer` class simply stores a list of upcoming events. It has a `call_after` method to add a new event. This method accepts a `delay` parameter representing the number of seconds to wait before executing the callback, and the `callback` function itself: a function to be executed at the correct time. This `callback` function should accept one argument.

The `run`  method is very simple; it uses a generator expression to filter out any  events whose time has come, and executes them in order. The **timer** loop then continues indefinitely, so it has to be interrupted with a keyboard interrupt (***Ctrl*** + **C,** or ***Ctrl*** + ***Break***). We sleep for half a second after each iteration so as to not grind the system to a halt.

The  important things to note here are the lines that touch callback  functions. The function is passed around like any other object and the  timer never knows or cares what the original name of the function is or  where it was defined. When it's time to call the function, the timer  simply applies the parenthesis syntax to the stored variable.

Here's a set of callbacks that test the timer:

```python
def format_time(message, *args):
    now = datetime.datetime.now()
    print(f"{now:%I:%M:%S}: {message}")


def one(timer):
    format_time("Called One")


def two(timer):
    format_time("Called Two")


def three(timer):
    format_time("Called Three")


class Repeater:
    def __init__(self):
        self.count = 0

    def repeater(self, timer):
        format_time(f"repeat {self.count}")
        self.count += 1
        timer.call_after(5, self.repeater)


timer = Timer()
timer.call_after(1, one)
timer.call_after(2, one)
timer.call_after(2, two)
timer.call_after(4, two)
timer.call_after(3, three)
timer.call_after(6, three)
repeater = Repeater()
timer.call_after(5, repeater.repeater)
format_time("Starting")
timer.run()
```

This example allows us to see how multiple callbacks interact with the timer. The first function is the `format_time` function. It uses the format string syntax to add the current time  to the message; we'll read about them in the next chapter. Next, we  create three simple callback methods that simply output the current time  and a short message telling us which callback has been fired.

The `Repeater`  class demonstrates that methods can be used as callbacks too, since  they are really just functions that happen to be bound to an object. It  also shows why the `timer` argument to the  callback functions is useful: we can add a new timed event to the timer  from inside a presently running callback. We then create a timer and add  several events to it that are called after different amounts of time.  Finally, we start the timer running; the output shows that events are  run in the expected order:

```python
02:53:35: Starting 
02:53:36: Called One 
02:53:37: Called One 
02:53:37: Called Two 
02:53:38: Called Three 
02:53:39: Called Two 
02:53:40: repeat 0 
02:53:41: Called Three 
02:53:45: repeat 1 
02:53:50: repeat 2 
02:53:55: repeat 3 
02:54:00: repeat 4
```

Python 3.4 introduced a generic event loop architecture similar to this. We'll be discussing it later, in **Concurrency**.

# Using Functions as Attributes

One of the interesting effects of functions being objects  is that they can be set as callable attributes on other objects. It is  possible to add or change a function to an instantiated object,  demonstrated as follows:

```python
class A: 
    def print(self): 
        print("my class is A") 
 
def fake_print(): 
    print("my class is not A") 
 
a = A() 
a.print() 
a.print = fake_print 
a.print() 
```

This code creates a very simple class with a `print` method that doesn't tell us anything we didn't know. Then, we create a new function that tells us something we don't believe.

When we call `print` on an instance of the `A` class, it behaves as expected. If we then set the `print` method to point at a new function, it tells us something different:

```python
my class is A 
my class is not A
```

It is also possible to replace methods on classes instead of objects, although, in that case, we have to add the `self`  argument to the parameter list. This will change the method for all  instances of that object, even ones that have already been instantiated.  Obviously, replacing methods  like this can be both dangerous and confusing to maintain. Somebody  reading the code will see that a method has been called and look up that  method on the original class. But the method on the original class is  not the one that was called. Figuring out what really happened can  become a tricky, frustrating debugging session.

It does have its uses though. Often, replacing or adding methods at runtime (called **monkey patching**)  is used in automated testing. If testing a client-server application,  we may not want to actually connect to the server while testing the  client; this may result in accidental transfers of funds or embarrassing  test emails being sent to real people. Instead, we can set up our test  code to replace some of the key methods on the object that sends  requests to the server so that it only records that the methods have  been called.

Monkey-patching can also be used to fix bugs or add  features in third-party code that we are interacting with, and does not  behave quite the way we need it to. It should, however, be applied  sparingly; it's almost always a **messy hack**. Sometimes, though, it is the only way to adapt an existing library to suit our needs.

# Callable Objects

Just as functions are objects that can have attributes set on them, it is possible to create an object that can be called as though it were a function.

Any object can be made callable by simply giving it a `__call__` method that accepts the required arguments. Let's make our `Repeater` class, from the timer example, a little easier to use by making it a callable, as follows:

```python
class Repeater: 
    def __init__(self): 
        self.count = 0 
 
 
 def __call__(self, timer): 
        format_time(f"repeat {self.count}") 
        self.count += 1 
 
        timer.call_after(5, self) 
 
timer = Timer() 
 
timer.call_after(5, Repeater()) 
format_time("{now}: Starting") 
timer.run() 
```

This example isn't much different from the earlier class; all we did was change the name of the `repeater` function to `__call__` and pass the object itself as a callable. Note that, when we make the `call_after` call, we pass the argument `Repeater()`. Those two parentheses are creating a new instance of the class; they are not explicitly calling the class. This happens later, inside the timer. If we want to execute the `__call__` method on a newly instantiated object, we'd use a rather odd syntax: `Repeater()()`. The first set of parentheses constructs the object; the second set executes the `__call__` method. If we find ourselves doing this, we may not be using the correct abstraction. Only implement the `__call__` function on an object if the object is meant to be treated like a function.