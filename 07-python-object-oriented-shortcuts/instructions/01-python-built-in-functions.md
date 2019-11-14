There are numerous functions  in Python that perform a task or calculate a result on certain types of  objects without being methods on the underlying class. They usually  abstract common calculations that apply to multiple types of classes.  This is duck typing at its best; these functions accept objects that  have certain attributes or methods, and are able to perform generic  operations using those methods. We've used many of the built-in  functions already, but let's quickly go through the important ones and  pick up a few neat tricks along the way.

# The len() Function

The simplest example is the `len()` function, which counts the number of items in some kind of container object, such as a dictionary or list. You've seen it before, demonstrated as follows:

```python
>>> len([1,2,3,4])4
```

You  may wonder why these objects don't have a length property instead of  having to call a function on them. Technically, they do. Most objects  that `len()` will apply to have a method called `__len__()` that returns the same value. So `len(myobj)` seems to call `myobj.__len__()`.

Why should we use the `len()` function instead of the `__len__` method? Obviously, `__len__`  is a special double-underscore method, suggesting that we shouldn't  call it directly. There must be an explanation for this. The Python  developers don't make such design decisions lightly.

The main reason is efficiency. When we call `__len__` on an object, the object has to look the method up in its namespace, and, if the special `__getattribute__`  method (which is called every time an attribute or method on an object  is accessed) is defined on that object, it has to be called as well.  Furthermore, the `__getattribute__` for that  particular method may have been written to do something nasty, such as  refusing to give us access to special methods such as `__len__`! The `len()` function doesn't encounter any of this. It actually calls the `__len__` function on the underlying class, so `len(myobj)` maps to `MyObj.__len__(myobj)`.

Another reason is maintainability. In the future, Python developers may want to change `len()` so that it can calculate the length of objects that don't have `__len__`, for example, by counting the number of items returned in an iterator. They'll only have to change one function instead of countless `__len__` methods in many objects across the board.

There is one other extremely important and often overlooked reason for `len()` being an external function: backward compatibility. This is often cited in articles as **for historical reasons**,  which is a mildly dismissive phrase that an author will use to say  something is the way it is because a mistake was made long ago and we're  stuck with it. Strictly speaking, `len()`  isn't a mistake, it's a design decision, but that decision was made in a  less object-oriented time. It has stood the test of time and has some  benefits, so do get used to it.

# Reversed

The `reversed()` function takes any sequence as input, and returns a copy of that sequence in reverse order. It is normally used in `for` loops when we want to loop over items from back to front.

Similar to `len`, `reversed` calls the `__reversed__()` function on the class for the parameter. If that method does not exist, `reversed` builds the reversed sequence itself using calls to `__len__` and `__getitem__`, which are used to define a sequence. We only need to override `__reversed__` if we want to somehow customize or optimize the process, as demonstrated in the following code:

```python
normal_list = [1, 2, 3, 4, 5]


class CustomSequence:
    def __len__(self):
        return 5

    def __getitem__(self, index):
        return f"x{index}"


class FunkyBackwards:
    def __reversed__(self):
        return "BACKWARDS!"


for seq in normal_list, CustomSequence(), FunkyBackwards():
    print(f"\n{seq.__class__.__name__}: ", end="")
```

```python
    for item in reversed(seq):
        print(item, end=", ")
```

The `for` loops at the end print reversed versions of a normal list, and instances of the two custom sequences. The output shows that `reversed` works on all three of them, but has very different results when we define `__reversed__` ourselves:

```python
list: 5, 4, 3, 2, 1,CustomSequence: x4, x3, x2, x1, x0,FunkyBackwards: B, A, C, K, W, A, R, D, S, !,
```

When we reverse `CustomSequence`, the `__getitem__` method is called for each item, which just inserts an `x` before the index. For `FunkyBackwards`, the `__reversed__` method returns a string, each character of which is output individually in the `for` loop.

info> The preceding two classes aren't very good sequences, as they don't define a proper version of `__iter__`, so a forward `for` loop over them would never end.

# Enumerate

Sometimes, when we're looping over a container in a `for` loop, we want access to the index (the current position in the list) of the current item being processed. The `for` loop doesn't provide us with indexes, but the `enumerate`  function gives us something better: it creates a sequence of tuples,  where the first object in each tuple is the index and the second is the  original item.

This is useful if we need to use index numbers  directly. Consider some simple code that outputs each of the lines in a  file with line numbers:

```python
import sys

filename = sys.argv[1]

with open(filename) as file:
    for index, line in enumerate(file):
        print(f"{index+1}: {line}", end="")
```

Running this code using its own filename as the input file shows how it works:

```python
1: import sys
2:
3: filename = sys.argv[1]
4:
5: with open(filename) as file:
6:     for index, line in enumerate(file):
7:         print(f"{index+1}: {line}", end="")
```

The `enumerate` function returns a sequence of tuples, our `for` loop splits each tuple into two values, and the `print` statement formats them together. It adds one to the index for each line number, since `enumerate`, like all sequences, is zero-based.

We've  only touched on a few of the more important Python built-in functions.  As you can see, many of them call into object-oriented concepts, while  others subscribe  to purely functional or procedural paradigms. There are numerous others  in the standard library; some of the more interesting ones include the  following:

- `all` and `any`, which accept an iterable object and return `True` if all, or any, of the items evaluate to true (such as a non-empty string or list, a non-zero number, an object that is not `None`, or the literal `True`).
- `eval`, `exec`, and `compile`,  which execute string as code inside the interpreter. Be careful with  these ones; they are not safe, so don't execute code an unknown user has  supplied to you (in general, assume all unknown users are malicious,  foolish, or both).
- `hasattr`, `getattr`, `setattr`, and `delattr`, which allow attributes on an object to be manipulated by their string names.
- `zip`,  which takes two or more sequences and returns a new sequence of tuples,  where each tuple contains a single value from each sequence.
- And many more! See the interpreter help documentation for each of the functions listed in `dir(__builtins__)`.

# File I/O

Our examples so far that have touched the filesystem have operated  entirely on text files without much thought as to what is going on  under the hood. Operating systems, however, actually represent files as a  sequence of bytes, not text. We'll take a deep dive into the  relationship between bytes and text in **Strings and Serialization**.  For now, be aware that reading textual data from a file is a fairly  involved process. Python, especially Python 3, takes care of most of  this work for us behind the scenes. Aren't we lucky?!

The concept of files has been around since long before anyone coined the term **object-oriented programming**.  However, Python has wrapped the interface that operating systems  provide in a sweet abstraction that allows us to work with file (or  file-like, vis-à-vis duck typing) objects.

The `open()`  built-in function is used to open a file and return a file object. For  reading text from a file, we only need to pass the name of the file into  the function. The file will be opened for reading, and the bytes will  be converted to text using the platform default encoding.

Of  course, we don't always want to read files; often we want to write data  to them! To open a file for writing, we need to pass a `mode` argument as the second positional argument, with a value of `"w"`:

```python
contents = "Some file contents" 
file = open("filename", "w") 
file.write(contents) 
file.close() 
```

We could also supply the value `"a"` as a mode argument, to append to the end of the file, rather than completely overwriting existing file content.

These  files with built-in wrappers for converting bytes to text are great,  but it'd be awfully inconvenient if the file we wanted to open was an  image, executable, or other binary file, wouldn't it?

To open a binary file, we modify the mode string to append `'b'`. So, `'wb'` would open a file for writing bytes, while `'rb'` allows us to read them. They will behave like text files, but without the automatic encoding of text to bytes. When we read such a file, it will return `bytes` objects instead of `str`, and when we write to it, it will fail if we try to pass a text object.

info> These  mode strings for controlling how files are opened are rather cryptic  and are neither Pythonic nor object-oriented. However, they are  consistent with virtually every other programming language out there.  File I/O is one of the fundamental jobs an operating system has to  handle, and all programming languages have to talk to the operating  system using the same system calls. Just be glad that Python returns a  file object with useful methods instead of the integer that most major  operating systems use to identify a file handle!

Once a file is opened for reading, we can call the `read`, `readline`, or `readlines` methods to get the contents of the file. The `read` method returns the entire contents of the file as a `str` or `bytes` object, depending on whether there is `'b'`  in the mode. Be careful not to use this method without arguments on  huge files. You don't want to find out what happens if you try to load  that much data into memory!

It is also possible to read a fixed number of bytes from a file; we pass an integer argument to the `read` method, describing how many bytes we want to read. The next call to `read` will load the next sequence of bytes, and so on. We can do this inside a `while` loop to read the entire file in manageable chunks.

The `readline`  method returns a single line from the file (where each line ends in a  newline, a carriage return, or both, depending on the operating system  on which the file was created). We can call it repeatedly to get  additional lines. The plural `readlines` method returns a list of all the lines in the file. Like the `read` method, it's not safe to use on very large files. These two methods even work when the file is open in `bytes`  mode, but it only makes sense if we are parsing text-like data that has  newlines at reasonable positions. An image or audio file, for example,  will not have newline characters in it (unless the newline byte happened  to represent a certain pixel or sound), so applying `readline` wouldn't make sense.

For readability, and to avoid reading a large file into memory at once, it is often better to use a `for`  loop directly on a file object. For text files, it will read each line,  one at a time, and we can process it inside the loop body. For binary  files, it's better to read fixed-sized chunks of data using the `read()` method, passing a parameter for the maximum number of bytes to read.

Writing to a file is just as easy; the `write`  method on file objects writes a string (or bytes, for binary data)  object to the file. It can be called repeatedly to write multiple  strings, one after the other. The `writelines` method accepts a sequence of strings and writes each of the iterated values to the file. The `writelines` method does *not*  append a new line after each item in the sequence. It is basically a  poorly named convenience function to write the contents of a sequence of  strings without having to explicitly iterate over it using a `for` loop.

Lastly, and I do mean lastly, we come to the `close`  method. This method should be called when we are finished reading or  writing the file, to ensure any buffered writes are written to the disk,  that the file has been properly cleaned up, and that all resources  associated with the file are released back to the operating system.  Technically, this will happen automatically when the script exits, but it's better to be explicit and clean up after ourselves, especially in long-running processes.

# Placing it in Context

The need to close files when we are finished with them can  make our code quite ugly. Because an exception may occur at any time  during file I/O, we ought to wrap all calls to a file in a `try`...`finally` clause. The file should be closed in the `finally` clause, regardless of whether I/O was successful. This isn't very Pythonic. Of course, there is a more elegant way to do it.

If we run `dir` on a file-like object, we see that it has two special methods named `__enter__` and `__exit__`. These methods turn the file object into what is known as a **context manager**. Basically, if we use a special syntax called the `with` statement, these methods will be called before and after nested code is executed. On file objects, the `__exit__`  method ensures the file is closed, even if an exception is raised. We  no longer have to explicitly manage the closing of the file. Here is  what the `with` statement looks like in practice:

```python
with open('filename') as file: 
    for line in file: 
        print(line, end='') 
```

The `open` call returns a file object, which has `__enter__` and `__exit__` methods. The returned object is assigned to the variable named `file` by the `as`  clause. We know the file will be closed when the code returns to the  outer indentation level, and that this will happen even if an exception  is raised.

The `with` statement is used  in several places in the standard library, where start up or cleanup  code needs to be executed. For example, the `urlopen` call returns an object that can be used in a `with`  statement to clean up the socket when we're done. Locks in the  threading module can automatically release the lock when the statement  has been executed.

Most interestingly, because the `with`  statement can apply to any object that has the appropriate special  methods, we can use it in our own frameworks. For example, remember that  strings are immutable, but sometimes you need to build a string from  multiple parts. For efficiency, this is usually done by storing the  component strings in a list and joining them at the end. Let's create  a simple context manager that allows us to construct a sequence of  characters and automatically convert it to a string upon exit:

```python
class StringJoiner(list): 
    def __enter__(self): 
        return self 
 
 def __exit__(self, type, value, tb): 
        self.result = "".join(self) 
```

This code adds the two special methods required of a context manager to the `list` class it inherits from. The `__enter__`  method performs any required setup code (in this case, there isn't any)  and then returns the object that will be assigned to the variable after  `as` in the `with` statement. Often, as we've done here, this is just the context manager object itself. The `__exit__` method accepts three arguments. In a normal situation, these are all given a value of `None`. However, if an exception occurs inside the `with` block, they will be set to values related to the type, value, and traceback for the exception. This allows the `__exit__`  method to perform any cleanup code that may be required, even if an  exception occurred. In our example, we take the irresponsible path and  create a result string by joining the characters in the string,  regardless of whether an exception was thrown.

While this is one of the simplest context managers we could write, and its usefulness is dubious, it does work with a `with` statement. Have a look at it in action:

```python
import random, string 
with StringJoiner() as joiner: 
    for i in range(15): 
        joiner.append(random.choice(string.ascii_letters)) 
 
print(joiner.result) 
```

This code constructs a string of 15 random characters. It appends these to a `StringJoiner` using the `append` method it inherited from `list`. When the `with` statement goes out of scope (back to the outer indentation level), the `__exit__` method is called, and the `result` attribute becomes available on the joiner object. We then print this value to see a random string.