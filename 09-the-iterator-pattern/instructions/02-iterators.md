In typical design pattern parlance, an iterator is an object with a `next()` method and a `done()` method; the latter returns `True`  if there are no items left in the sequence. In a programming language  without built-in support for iterators, the iterator would be looped  over like this:

```python
while not iterator.done(): 
    item = iterator.next() 
    # do something with the item 
```

In Python, iteration is a special feature, so the method gets a special name, `__next__`. This method can be accessed using the `next(iterator)` built-in. Rather than a `done` method, Python's iterator protocol raises `StopIteration` to notify the loop that it has completed. Finally, we have the much more readable `foriteminiterator` syntax to actually access items in an iterator instead of messing around with a `while` loop. Let's look at these in more detail.

# The Iterator Protocol

The `Iterator` abstract base class, in the `collections.abc` module, defines the iterator protocol in Python. As mentioned, it must have a `__next__` method that the `for`  loop (and other features that support iteration) can call to get a new  element from the sequence. In addition, every iterator must also fulfill  the `Iterable` interface. Any class that provides an `__iter__` method is iterable. That method must return an `Iterator` instance that will cover all the elements in that class.

This  might sound a bit confusing, so have a look at the following example,  but note that this is a very verbose way to solve this problem. It  clearly explains iteration and the two protocols in question, but we'll  be looking at several more readable ways to get this effect later in  this lesson:

```python
class CapitalIterable: 
    def __init__(self, string): 
        self.string = string 
 
    def __iter__(self): 
        return CapitalIterator(self.string) 
 
 
class CapitalIterator: 
    def __init__(self, string): 
        self.words = [w.capitalize() for w in string.split()] 
        self.index = 0 
 
    def __next__(self): 
        if self.index == len(self.words): 
            raise StopIteration() 
 
        word = self.words[self.index] 
        self.index += 1 
        return word 
 
    def __iter__(self): 
        return self 
```

This example defines an `CapitalIterable`  class whose job is to loop over each of the words in a string and  output them with the first letter capitalized. Most of the work of that iterable is passed to the `CapitalIterator` implementation. The canonical way to interact with this iterator is as follows:

```python
>>> iterable = CapitalIterable('the quick brown fox jumps over the lazy dog')>>> iterator = iter(iterable)>>> while True:...     try:...         print(next(iterator))...     except StopIteration:...         break...     TheQuickBrownFoxJumpsOverTheLazyDog
```

This  example first constructs an iterable and retrieves an iterator from it.  The distinction may need explanation; the iterable is an object with  elements that can be looped over. Normally, these elements can be looped  over multiple times, maybe even at the same time or in overlapping  code. The iterator, on the other hand, represents a specific location in  that iterable;  some of the items have been consumed and some have not. Two different  iterators might be at different places in the list of words, but any one  iterator can mark only one place.

Each time `next()`  is called on the iterator, it returns another token from the iterable,  in order. Eventually, the iterator will be exhausted (won't have any  more elements to return), in which case `Stopiteration` is raised, and we break out of the loop.

Of course, we already know a much simpler syntax for constructing an iterator from an iterable:

```python
>>> for i in iterable:...     print(i)...     TheQuickBrownFoxJumpsOverTheLazyDog
```

As you can see, the `for`  statement, in spite of not looking remotely object-oriented, is  actually a shortcut to some obviously object-oriented design principles.  Keep this in mind as we discuss comprehensions, as they, too, appear to  be the polar opposite of an object-oriented tool. Yet, they use the exact same iteration protocol as `for` loops and are just another kind of shortcut.

