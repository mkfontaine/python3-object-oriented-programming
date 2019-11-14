Coroutines are extremely powerful constructs that are often confused with generators. Many authors inappropriately describe coroutines as **generators with a bit of extra syntax**. This is an easy mistake to make, as, way back in Python 2.5, when coroutines were introduced, they were presented as **we added a** `send` **method to the generator syntax**. The difference is actually a lot more nuanced and will make more sense after you've seen a few examples.

Coroutines are pretty hard to understand. Outside the `asyncio`  module, which we'll cover in the chapter on concurrency, they are not  used all that often in the wild. You can definitely skip this section  and happily develop in Python for years without ever encountering  coroutines. There are a couple of libraries that use coroutines  extensively (mostly for concurrent or asynchronous programming), but  they are normally written such that you can use coroutines without  actually understanding how they work! So, if you get lost in this  section, don't despair.

If I haven't scared you off, let's get  started! Here's one of the simplest possible coroutines; it allows us to  keep a running tally that can be increased by arbitrary values:

```python
def tally(): 
    score = 0 
    while True: 
        increment = yield score 
        score += increment 
```

This code looks like  black magic that couldn't possibly work, so let's prove it works before  going into a line-by-line description. This simple object could be used  by a scoring application for a baseball team. Separate tallies could be  kept for each team, and their score could be incremented by the number of runs accumulated at the end of every half-innings. Look at this interactive session:

```python
>>> white_sox = tally()>>> blue_jays = tally()>>> next(white_sox)0>>> next(blue_jays)0>>> white_sox.send(3)3>>> blue_jays.send(2)2>>> white_sox.send(2)5>>> blue_jays.send(4)6
```

First, we construct two `tally`  objects, one for each team. Yes, they look like functions, but as with  the generator objects in the previous section, the fact that there is a `yield` statement inside the function tells Python to put a great deal of effort into turning the simple function into an object.

We then call `next()`  on each of the coroutine objects. This does the same thing as calling  next on any generator, which is to say, it executes each line of code  until it encounters a `yield` statement, returns the value at that point, and then *pauses* until the next `next()` call.

So far, then, there's nothing new. But look back at the `yield` statement in our coroutine:

```python
increment = yield score 
```

Unlike with generators, this `yield` function looks  like it's supposed to return a value and assign it to a variable. In  fact, this is exactly what's happening. The coroutine is still paused at  the `yield` statement and waiting to be activated again by another call to `next()`.

Except we don't call `next()`. As you see in the interactive session, we instead call to a method called `send()`. The `send()` method does **exactly** the same thing as `next()` except that in addition to advancing the generator to the next `yield`  statement, it also allows you to pass in a value from outside the  generator. This value is what gets assigned to the left side of the `yield` statement.

The thing that is really confusing for many people is the order in which this happens:

1. `yield` occurs and the generator pauses
2. `send()` occurs from outside the function and the generator wakes up
3. The value sent in is assigned to the left side of the `yield` statement
4. The generator continues processing until it encounters another `yield` statement

So, in this particular example, after we construct the coroutine and advance it to the `yield` statement with a single call to `next()`, each successive call to `send()` passes a value into the coroutine. We add this value to its score. Then we go back to the top of the `while` loop, and keep processing until we hit the `yield` statement. The `yield` statement returns a value, which becomes the return value of our most recent call to `send`. Don't miss that: like `next()`, the `send()` method does not just submit a value to the generator, it also returns the value from the upcoming `yield` statement. This is how we define the difference between a generator and a coroutine: a generator only produces values, while a coroutine can also consume them.

info> The behavior and syntax of `next(i)`, `i.__next__()`, and `i.send(value)`  are rather unintuitive and frustrating. The first is a normal function,  the second is a special method, and the last is a normal method. But  all three do the same thing: advance the generator until it yields a  value and pause. Further, the `next()` function and associated method can be replicated by calling `i.send(None)`.  There is value to having two different method names here, since it  helps the reader of our code easily see whether they are interacting  with a coroutine or a generator. I just find the fact that in one case  it's a function call and in the other it's a normal method somewhat  irritating.

# Back to Log Parsing

Of course, the previous example could easily have been coded using a couple of integer variables and calling `x += increment`  on them. Let's look at a second example where coroutines actually save  us some code. This example is a somewhat simplified (for pedagogical  reasons) version of a problem I had to solve while working at Facebook.

The Linux kernel log contains lines that look almost, but not quite entirely, unlike this:

```markdown
unrelated log messages 
sd 0:0:0:0 Attached Disk Drive 
unrelated log messages 
sd 0:0:0:0 (SERIAL=ZZ12345) 
unrelated log messages 
sd 0:0:0:0 [sda] Options 
unrelated log messages 
XFS ERROR [sda] 
unrelated log messages 
sd 2:0:0:1 Attached Disk Drive 
unrelated log messages 
sd 2:0:0:1 (SERIAL=ZZ67890) 
unrelated log messages 
sd 2:0:0:1 [sdb] Options 
unrelated log messages 
sd 3:0:1:8 Attached Disk Drive 
unrelated log messages 
sd 3:0:1:8 (SERIAL=WW11111) 
unrelated log messages 
sd 3:0:1:8 [sdc] Options 
unrelated log messages 
XFS ERROR [sdc] 
unrelated log messages
```

There are a  whole bunch of interspersed kernel log messages, some of which pertain  to hard disks. The hard disk messages might be interspersed with other  messages, but they occur in a predictable format and order. For each, a  specific drive with a known serial number is associated with a bus identifier (such as `0:0:0:0`). A block device identifier (such as `sda`) is also associated with that bus. Finally, if the drive has a corrupt filesystem, it might fail with an XFS error.

Now,  given the preceding log file, the problem we need to solve is how to  obtain the serial number of any drives that have XFS errors on them.  This serial number might later be used by a data center technician to  identify and replace the drive.

We know we can identify the  individual lines using regular expressions, but we'll have to change the  regular expressions as we loop through the lines, since we'll be  looking for different things depending on what we found previously. The  other difficult bit is that if we find an error string, the information  about which bus contains that string as well as the serial number have  already been processed. This can easily be solved by iterating through  the lines of the file in reverse order.

Before you look at this example, be warned—the amount of code required for a coroutine-based solution is scarily small:

```python
import re


def match_regex(filename, regex):
    with open(filename) as file:
        lines = file.readlines()
    for line in reversed(lines):
        match = re.match(regex, line)
        if match:
            regex = yield match.groups()[0]


def get_serials(filename):
    ERROR_RE = "XFS ERROR (\[sd[a-z]\])"
    matcher = match_regex(filename, ERROR_RE)
    device = next(matcher)
    while True:
        try:
            bus = matcher.send(
                "(sd \S+) {}.*".format(re.escape(device))
            )
            serial = matcher.send("{} \(SERIAL=([^)]*)\)".format(bus))
 yield serial
            device = matcher.send(ERROR_RE)
        except StopIteration:
            matcher.close()
            return


for serial_number in get_serials("EXAMPLE_LOG.log"):
    print(serial_number)
```

This code neatly divides the job into two separate  tasks. The first task is to loop over all the lines and spit out any  lines that match a given regular expression. The second task is to  interact with the first task and give it guidance as to what regular  expression it is supposed to be searching for at any given time.

Look at the `match_regex`  coroutine first. Remember, it doesn't execute any code when it is  constructed; rather, it just creates a coroutine object. Once  constructed, someone outside the coroutine will eventually call `next()` to start the code running. Then it stores the state of two variables `filename` and `regex`.  It then reads all the lines in the file and iterates over them in  reverse. Each line is compared to the regular expression that was passed  in until it finds a match. When the match is found, the coroutine  yields the first group from the regular expression and waits.

At  some point in the future, other code will send in a new regular  expression to search for. Note that the coroutine never cares what  regular expression it is trying to match; it's just looping over lines  and comparing them to a regular expression. It's somebody else's  responsibility to decide what regular expression to supply.

In this case, that somebody else is the `get_serials`  generator. It doesn't care about the lines in the file; in fact, it  isn't even aware of them. The first thing it does is create a `matcher` object from the `match_regex` coroutine constructor, giving it a default regular expression to search for. It advances the coroutine to its first `yield` and stores the value it returns. It then goes into a loop that instructs the `matcher` object to search for a bus ID based on the stored device ID, and then a serial number based on that bus ID.

It idly yields that serial number to the outside `for` loop before instructing the matcher to find another device ID and repeat the cycle.

Basically, the coroutine's job is to search for the next important line in the file, while the generator's (`get_serial`, which uses the `yield`  syntax without assignment) job is to decide which line is important.  The generator has information about this particular problem, such as  what order lines will appear in the file. The coroutine, on the other  hand, could be plugged into any problem that required searching a file  for given regular expressions.

# Closing Coroutines and Throwing Exceptions

Normal generators signal their exit from inside by raising `StopIteration`. If we chain multiple generators together (for example, by iterating over one generator from inside another), the `StopIteration` exception will be propagated outward. Eventually, it will hit a `for` loop that will see the exception and know that it's time to exit the loop.

Even  though they use a similar syntax, coroutines don't normally follow the  iteration mechanism. Instead of pulling data through one until an  exception is encountered, data is usually pushed into it (using `send`).  The entity doing the pushing is normally the one in charge of telling  the coroutine when it's finished. It does this by calling the `close()` method on the coroutine in question.

When called, the `close()` method will raise a `GeneratorExit`  exception at the point the coroutine was waiting for a value to be sent  in. It is normally good policy for coroutines to wrap their `yield` statements in a `try`...`finally` block so that any cleanup tasks (such as closing associated files or sockets) can be performed.

If we need to raise an exception inside a coroutine, we can use the `throw()` method in a similar way. It accepts an exception type with optional `value` and `traceback` arguments. The latter is useful when we encounter an exception in one coroutine and want to cause an exception to occur in an adjacent coroutine while maintaining the traceback.

info> The  previous example could be written without coroutines and would be about  equally readable. The truth is, correctly managing all the state  between coroutines is pretty difficult, especially when you take things  like context managers and exceptions into account. Luckily, the Python  standard library contains a package called `asyncio`  that can manage all of this for you. We'll cover that in the chapter on  concurrency. In general, I recommend you avoid using bare coroutines  unless you are specifically coding for asyncio. The logging example  could almost be considered an *anti-pattern*; a design pattern that should be avoided rather than embraced.

# The Relationship Between Coroutines, Generators, and Functions

We've seen coroutines in action, so now let's go back to that discussion of how they are related to generators. In Python, as is so often the case, the distinction  is quite blurry. In fact, all coroutines are generator objects, and  authors often use the two terms interchangeably. Sometimes, they  describe coroutines as a subset of generators (only generators that  return values from yield are considered coroutines). This is technically  true in Python, as we've seen in the previous sections.

However,  in the greater sphere of theoretical computer science, coroutines are  considered the more general principles, and generators are a specific  type of coroutine. Further, normal functions are yet another distinct  subset of coroutines.

A coroutine is a routine that can have data  passed in at one or more points and get it out at one or more points. In  Python, the point where data is passed in and out is the `yield` statement.

A  function, or subroutine, is the simplest type of coroutine. You can  pass data in at one point, and get data out at one other point when the  function returns. While a function can have multiple `return` statements, only one of them can be called for any given invocation of the function.

Finally,  a generator is a type of coroutine that can have data passed in at one  point, but can pass data out at multiple points. In Python, the data  would be passed out at a `yield` statement, but you can't pass data back in. If you called `send`, the data would be silently discarded.

So,  in theory, generators are types of coroutines, functions are types of  coroutines, and there are coroutines that are neither functions nor  generators. That's simple enough, eh? So, why does it feel more  complicated in Python?

In Python, generators and coroutines are both constructed using a syntax that **looks**  like we are constructing a function. But the resulting object is not a  function at all; it's a totally different kind of object. Functions are,  of course, also objects. But they have a different interface; functions  are callable and return values, generators have data pulled out using `next()`, and coroutines have data pushed in using `send`.

info> There is an alternate syntax for coroutines using the `async` and `await` keywords. The  syntax makes it clearer that the code is a coroutine and further breaks  the deceiving symmetry between coroutines and generators. The syntax  doesn't work very well without building a full event loop, so we will  skip it until we cover `asyncio` in the concurrency lesson.