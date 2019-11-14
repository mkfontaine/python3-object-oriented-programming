Generator expressions are actually  a sort of comprehension too; they compress the more advanced (this time  it really is more advanced!) generator syntax into one line. The  greater generator syntax looks even less object-oriented than anything  we've seen, but we'll discover that once again, it is a simple syntax  shortcut to create a kind of object.

Let's take the log file example a little further. If we want to delete the `WARNING`  column from our output file (since it's redundant: this file contains  only warnings), we have several options at various levels of  readability. We can do it with a generator expression:

```python
import sys

# generator expression
inname, outname = sys.argv[1:3]

with open(inname) as infile:
    with open(outname, "w") as outfile:
        warnings = (
            l.replace("\tWARNING", "") for l in infile if "WARNING" in l
        )
        for l in warnings:
            outfile.write(l)
```

That's perfectly  readable, though I wouldn't want to make the expression much more  complicated than that. We could also do it with a normal `for` loop:

```python
with open(inname) as infile:
    with open(outname, "w") as outfile:
        for l in infile:
            if "WARNING" in l:
                outfile.write(l.replace("\tWARNING", ""))
```

That's  clearly maintainable, but so many levels of indent in so few lines is  kind of ugly. More alarmingly, if we wanted to do something other than printing the lines out, we'd have to duplicate the looping and conditional code, too.

Now let's consider a truly object-oriented solution, without any shortcuts:

```python
class WarningFilter:
    def __init__(self, insequence):
        self.insequence = insequence

    def __iter__(self):
        return self

    def __next__(self):
        l = self.insequence.readline()
        while l and "WARNING" not in l:
            l = self.insequence.readline()
        if not l:
 raise StopIteration
        return l.replace("\tWARNING", "")


with open(inname) as infile:
    with open(outname, "w") as outfile:
        filter = WarningFilter(infile)
        for l in filter:
            outfile.write(l)
```

No doubt about it:  that is so ugly and difficult to read that you may not even be able to  tell what's going on. We created an object that takes a file object as  input, and provides a `__next__` method like any iterator.

This `__next__` method reads lines from the file, discarding them if they are not `WARNING` lines. When we encounter a `WARNING` line, we modify and return it. Then our `for` loop calls `__next__` again to process the subsequent `WARNING` line. When we run out of lines, we raise `StopIteration` to tell the loop we're finished iterating. It's pretty ugly compared to the other examples, but it's also powerful; now that we have a class in our hands, we can do whatever we want with it.

With that background behind us, we finally get to see true generators in action. This next example does **exactly** the same thing as the previous one: it creates an object with a `__next__` method that raises `StopIteration` when it's out of inputs:

```python
def warnings_filter(insequence):
    for l in insequence:
        if "WARNING" in l:
            yield l.replace("\tWARNING", "")


with open(inname) as infile:
    with open(outname, "w") as outfile:
        filter = warnings_filter(infile)
        for l in filter:
            outfile.write(l)
```

OK, that's pretty  readable, maybe... at least it's short. But what on earth is going on  here? It makes no sense whatsoever. And what is `yield`, anyway?

In fact, `yield` is the key to generators. When Python sees `yield` in a function, it takes that function and wraps it up in an object not unlike the one in our previous example. Think of the `yield` statement as similar to the `return` statement; it exits the function and returns a line. Unlike `return`, however, when the function is called again (via `next()`), it will start where it left off—on the line after the `yield` statement—instead of at the beginning of the function. In this example, there is no line *after* the `yield` statement, so it jumps to the next iteration of the `for` loop. Since the `yield` statement is inside an `if` statement, it only yields lines that contain `WARNING`.

While it looks like this is just a function looping over the lines, it is actually creating a special type of object, a generator object:

```python
>>> print(warnings_filter([]))<generator object warnings_filter at 0xb728c6bc>
```

I  passed an empty list into the function to act as an iterator. All the  function does is create and return a generator object. That object has `__iter__` and `__next__` methods on it, just like the one we created in the previous example. (You can call the `dir` built-in function on it to confirm.) Whenever `__next__` is called, the generator runs the function until it finds a `yield` statement. It then returns the value from `yield`, and the next time `__next__` is called, it picks up where it left off.

This  use of generators isn't that advanced, but if you don't realize the  function is creating an object, it can seem like magic. This example was  quite simple, but you can get really powerful effects by making  multiple calls to `yield` in a single function; on each loop, the generator will simply pick up at the most recent `yield` and continue to the next one.

# Yield Items from Another Iterable

Often, when we build a generator  function, we end up in a situation where we want to yield data from  another iterable object, possibly a list comprehension or generator  expression we constructed inside the generator, or perhaps some external  items that were passed into the function. This has always been possible  by looping over the iterable and individually yielding each item.  However, in Python version 3.3, the Python developers introduced a new  syntax to make it a little more elegant.

Let's adapt the generator  example a bit so that instead of accepting a sequence of lines, it  accepts a filename. This would normally be frowned upon as it ties the  object to a particular paradigm. When possible we should operate on  iterators as input; this way the same function could be used regardless  of whether the log lines came from a file, memory, or the web.

This  version of the code illustrates that your generator can do some basic  setup before yielding information from another iterable (in this case, a  generator expression):

```python
def warnings_filter(infilename):
    with open(infilename) as infile:
        yield from (
            l.replace("\tWARNING", "") for l in infile if "WARNING" in l
        )


filter = warnings_filter(inname)
with open(outname, "w") as outfile:
    for l in filter:
        outfile.write(l)
```

This code combines the `for` loop from the previous example into a generator expression. Notice that this transformation didn't help anything; the previous example with a `for` loop was more readable.

So,  let's consider an example that is more readable than its alternative.  It can be useful to construct a generator that yields data from multiple  other generators. The `itertools.chain`  function, for example, yields data from iterables in sequence until they  have all been exhausted. This can be implemented far too easily using  the `yield from` syntax, so let's consider a classic computer science problem: walking a general tree.

A  common implementation of the general tree data structure is a  computer's filesystem. Let's model a few folders and files in a Unix  filesystem so we can use `yield from` to walk them effectively:

```python
class File:
    def __init__(self, name):
        self.name = name


class Folder(File):
    def __init__(self, name):
        super().__init__(name)
        self.children = []


root = Folder("")
etc = Folder("etc")
root.children.append(etc)
etc.children.append(File("passwd"))
etc.children.append(File("groups"))
httpd = Folder("httpd")
etc.children.append(httpd)
httpd.children.append(File("http.conf"))
var = Folder("var")
root.children.append(var)
log = Folder("log")
var.children.append(log)
log.children.append(File("messages"))
log.children.append(File("kernel"))
```

This setup code looks like a lot of work, but in a  real filesystem, it would be even more involved. We'd have to read data  from the hard drive and structure it into the tree. Once in memory,  however, the code that outputs every file in the filesystem is quite  elegant:

```python
def walk(file):
    if isinstance(file, Folder):
        yield file.name + "/"
        for f in file.children:
            yield from walk(f)
    else:
        yield file.name
```

If this code encounters a directory, it recursively asks `walk()`  to generate a list of all files subordinate to each of its children,  and then yields all that data plus its own filename. In the simple case  that it has encountered a normal file, it just yields that name.

As an aside, solving the preceding problem without using a generator  is tricky enough that it is a common interview question. If you answer  it as shown like this, be prepared for your interviewer to be both  impressed and somewhat irritated that you answered it so easily. They  will likely demand that you explain exactly what is going on. Of course,  armed with the principles you've learned in this chapter, you won't  have any problem. Good luck!

The `yield from`  syntax is a useful shortcut when writing chained generators. It was  added to the language for a different reason, to support coroutines. It  is not used all that much anymore, however, because it's usage has been  replaced with `async` and `await` syntax. We'll see examples of both in the next section.