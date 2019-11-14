Comprehensions are simple, but powerful, syntaxes that allow us to  transform or filter an iterable object in as little as one line of code.  The resultant  object can be a perfectly normal list, set, or dictionary, or it can be a  generator expression that can be efficiently consumed while keeping  just one element in memory at a time.

# List Comprehensions

List comprehensions are one of the most powerful tools in Python, so people tend to think of them as advanced. They're not. Indeed, I've taken the liberty of littering previous  examples with comprehensions, assuming you would understand them. While  it's true that advanced programmers use comprehensions a lot, it's not  because they're advanced. It's because they're trivial, and handle some  of the most common operations in software development.

Let's have a  look at one of those common operations; namely, converting a list of  items into a list of related items. Specifically, let's assume we just  read a list of strings from a file, and now we want to convert it to a  list of integers. We know every item in the list is an integer, and we  want to do some activity (say, calculate an average) on those numbers.  Here's one simple way to approach it:

```python
input_strings = ["1", "5", "28", "131", "3"]
 
output_integers = [] 
for num in input_strings: 
    output_integers.append(int(num)) 
```

This works  fine and it's only three lines of code. If you aren't used to  comprehensions, you may not even think it looks ugly! Now, look at the  same code using a list comprehension:

```python
input_strings = ["1", "5", "28", "131", "3"]
output_integers = [int(num) for num in input_strings]
```

We're down to one line and, importantly for performance, we've dropped an `append`  method call for each item in the list. Overall, it's pretty easy to  tell what's going on, even if you're not used to comprehension syntax.

The square brackets indicate, as always, that we're creating a list. Inside this list is a `for` loop that iterates over each item in the input sequence. The only thing that may be confusing is what's happening between the list's opening brace and the start of the `for` loop. Whatever happens here is applied to *each* of the items in the input list. The item in question is referenced by the `num` variable from the loop. So, it's calling the `int` function for each element and storing the resulting integer in the new list.

That's  all there is to a basic list comprehension. Comprehensions are highly  optimized C code; list comprehensions are far faster than `for`  loops when looping over a large number of items. If readability alone  isn't a convincing reason to use them as much as possible, speed should  be.

Converting one list of items into a related list isn't the  only thing we can do with a list comprehension. We can also choose to  exclude certain values by adding an `if` statement inside the comprehension. Have a look:

```python
output_integers = [int(num) for num in input_strings if len(num) < 3]
```

All that's different between this example and the previous one is the `if len(num) < 3` part. This extra code excludes any strings with more than two characters. The `if` statement is applied to each element **before** the `int`  function, so it's testing the length of a string. Since our input  strings are all integers at heart, it excludes any number over 99.

List  comprehensions are used to map input values to output values, applying a  filter along the way to include or exclude any values that meet a  specific condition.

Any iterable can be the input to a list comprehension. In other words, anything we can wrap in a `for` loop can also be placed inside a comprehension. For example, text files are iterable; each call to `__next__` on the file's iterator will return one line of the file. We could load a tab-delimited file where the first line is a header row into a dictionary using the `zip` function:

```python
import sys

filename = sys.argv[1]

with open(filename) as file:
    header = file.readline().strip().split("\t")
 contacts = [
 dict(
 zip(header, line.strip().split("\t")))
 for line in file
 ]

for contact in contacts:
    print("email: {email} -- {last}, {first}".format(**contact))
```

This time, I've added some whitespace to make it more readable (list comprehensions don't **have**  to fit on one line). This example creates a list of dictionaries from  the zipped header and split lines for each line in the file.

Er,  what? Don't worry if that code or explanation doesn't make sense; it's  confusing. One list comprehension is doing a pile of work here, and the  code is hard to understand, read, and ultimately, maintain. This example  shows that list comprehensions aren't always the best solution; most programmers would agree that a `for` loop would be more readable than this version.

info> Remember:  the tools we are provided with should not be abused! Always pick the  right tool for the job, which is always to write maintainable code.

# Set and Dictionary Comprehensions

Comprehensions aren't restricted to lists. We can use a similar syntax with  braces to create sets and dictionaries as well. Let's start with sets.  One way to create a set is to wrap a list comprehension in the `set()`  constructor, which converts it to a set. But why waste memory on an  intermediate list that gets discarded, when we can create a set  directly?

Here's an example that uses a named tuple to model  author/title/genre triads, and then retrieves a set of all the authors  that write in a specific genre:

```python
from collections import namedtuple

Book = namedtuple("Book", "author title genre")
books = [
    Book("Pratchett", "Nightwatch", "fantasy"),
    Book("Pratchett", "Thief Of Time", "fantasy"),
    Book("Le Guin", "The Dispossessed", "scifi"),
    Book("Le Guin", "A Wizard Of Earthsea", "fantasy"),
    Book("Turner", "The Thief", "fantasy"),
    Book("Phillips", "Preston Diamond", "western"),
    Book("Phillips", "Twice Upon A Time", "scifi"),
]

fantasy_authors = {b.author for b in books if b.genre == "fantasy"}
```

The highlighted set comprehension sure is short in  comparison to the demo-data setup! If we were to use a list  comprehension, of course, Terry Pratchett would have been listed twice. As it is, the nature of sets removes the duplicates, and we end up with the following:

```python
>>> fantasy_authors{'Turner', 'Pratchett', 'Le Guin'}
```

Still using braces, we can introduce a colon to create a dictionary comprehension. This converts a sequence into a dictionary using **key:value** pairs. For example, it may be useful to quickly look up the author or genre in a dictionary if we know the title. We can use a dictionary comprehension to map titles to `books` objects:

```python
fantasy_titles = {b.title: b for b in books if b.genre == "fantasy"}
```

Now, we have a dictionary, and can look up books by title using the normal syntax.

In summary, comprehensions are not advanced Python, nor are they *non-object-oriented* tools  that should be avoided. They are simply a more concise and optimized  syntax for creating a list, set, or dictionary from an existing  sequence.

# Generator Expressions

Sometimes we want to process a new sequence without pulling  a new list, set, or dictionary into system memory. If we're just  looping over items one at a time, and don't actually care about having a  complete container (such as a list or dictionary) created, creating  that container is a waste of memory. When processing one item at a time,  we only need the current object available in memory at any one moment.  But when we create a container, all the objects have to be stored in  that container before we start processing them.

For example, consider a program that processes log files. A very simple log might contain information in this format:

```python
Jan 26, 2015 11:25:25 DEBUG This is a debugging message. Jan 26, 2015 11:25:36 INFO This is an information method. Jan 26, 2015 11:25:46 WARNING This is a warning. It could be serious. Jan 26, 2015 11:25:52 WARNING Another warning sent. Jan 26, 2015 11:25:59 INFO Here's some information. Jan 26, 2015 11:26:13 DEBUG Debug messages are only useful if you want to figure something out. Jan 26, 2015 11:26:32 INFO Information is usually harmless, but helpful. Jan 26, 2015 11:26:40 WARNING Warnings should be heeded. Jan 26, 2015 11:26:54 WARNING Watch for warnings.
```

Log  files for popular web servers, databases, or email servers can contain  many gigabytes of data (I once had to clean nearly 2 terabytes of logs  off a misbehaving system). If we want to process each line in the log,  we can't use a list comprehension; it would create a list containing  every line in the file. This probably wouldn't fit in RAM and could  bring the computer to its knees, depending on the operating system.

If we used a `for`  loop on the log file, we could process one line at a time before  reading the next one into memory. Wouldn't be nice if we could use  comprehension syntax to get the same effect?

This is where  generator expressions come in. They use the same syntax as  comprehensions, but they don't create a final container object. To create a generator expression, wrap the comprehension in `()` instead of `[]` or `{}`.

The following code parses a log file in the previously presented format and outputs a new log file that contains only the `WARNING` lines:

```python
import sys 
 
inname = sys.argv[1] 
outname = sys.argv[2] 
 
with open(inname) as infile: 
    with open(outname, "w") as outfile: 
        warnings = (l for l in infile if 'WARNING' in l) 
        for l in warnings: 
            outfile.write(l) 
```

This program takes  the two filenames on the command line, uses a generator expression to  filter out the warnings (in this case, it uses the `if`  syntax and leaves the line unmodified), and then outputs the warnings  to another file. If we run it on our sample file, the output looks like  this:

```python
Jan 26, 2015 11:25:46 WARNING This is a warning. It could be serious.
Jan 26, 2015 11:25:52 WARNING Another warning sent.
Jan 26, 2015 11:26:40 WARNING Warnings should be heeded.
Jan 26, 2015 11:26:54 WARNING Watch for warnings.
```

Of  course, with such a short input file, we could have safely used a list  comprehension, but if the file is millions of lines long, the generator  expression will have a huge impact on both memory and speed.

info> Wrapping a `for` expression in parenthesis creates a generator expression, not a tuple.

Generator expressions are frequently most useful inside function calls. For example, we can call `sum`, `min`, or `max` on  a generator expression instead of a list, since these functions process  one object at a time. We're only interested in the aggregate result,  not any intermediate container.

In  general, of the four options, a generator expression should be used  whenever possible. If we don't actually need a list, set, or dictionary,  but simply need to filter or convert items in a sequence, a generator  expression will be most efficient. If we need to know the length of a  list, or sort the result, remove duplicates, or create a dictionary,  we'll have to use the comprehension syntax.