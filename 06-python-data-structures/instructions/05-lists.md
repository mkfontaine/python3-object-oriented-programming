Lists are the least object-oriented of Python's data structures.  While lists are, themselves, objects, there is a lot of syntax in Python  to make using them as painless  as possible. Unlike many other object-oriented languages, lists in  Python are simply available. We don't need to import them and rarely  need to call methods on them. We can loop over a list without explicitly  requesting an iterator object, and we can construct a list (as with a  dictionary) with custom syntax. Further, list comprehensions and  generator expressions turn them into a veritable Swiss Army knife of  computing functionality.

We won't go into too much detail of the  syntax; you've seen it in introductory tutorials across the web and in  previous examples in this book. You can't code Python for very long  without learning how to use lists! Instead, we'll be covering when lists  should be used, and their nature as objects. If you don't know how to  create or append to a list, how to retrieve items from a list, or what **slice notation** is, I direct you to the official Python tutorial, posthaste. It can be found online [here](<http://docs.python.org/3/tutorial/>).

In Python, lists should normally be used when we want to store several instances of the **same** type  of object; lists of strings or lists of numbers; most often, lists of  objects we've defined ourselves. Lists should always be used when we  want to store items in some kind of order. Often, this is the order in  which they were inserted, but they can also be sorted by other criteria.

As  we saw in the case study from the previous chapter, lists are also very  useful when we need to modify the contents: insert to, or delete from,  an arbitrary location of the list, or update a value within the list.

Like  dictionaries, Python lists use an extremely efficient and well-tuned  internal data structure so we can worry about what we're storing, rather  than how we're storing  it. Many object-oriented languages provide different data structures  for queues, stacks, linked lists, and array-based lists. Python does  provide special instances of some of these classes, if optimizing access  to huge sets of data is required. Normally, however, the list data  structure can serve all these purposes at once, and the coder has  complete control over how they access it.

Don't use lists for  collecting different attributes of individual items. We do not want, for  example, a list of the properties a particular shape has. Tuples, named  tuples, dictionaries, and objects would all be more suitable for this  purpose. In some languages, they might create a list in which each  alternate item is a different type; for example, they might write `['a', 1, 'b', 3]`  for our letter frequency list. They'd have to use a strange loop that  accesses two elements in the list at once or a modulus operator to  determine which position was being accessed.

Don't do this in  Python. We can group related items together using a dictionary, as we  did in the previous section, or using a list of tuples. Here's a rather  convoluted counter-example that demonstrates how we could perform the  frequency example using a list. It is much more complicated than the  dictionary examples, and illustrates the effect choosing the right (or  wrong) data structure can have on the readability of our code. This is  demonstrated as follows:

```python
import string 
CHARACTERS  = list(string.ascii_letters) + [" "] 
 
def letter_frequency(sentence): 
    frequencies = [(c, 0) for c in CHARACTERS] 
    for letter in sentence: 
        index = CHARACTERS.index(letter) 
        frequencies[index] = (letter,frequencies[index][1]+1) 
    return frequencies 
```

This code starts with a list of possible characters. The `string.ascii_letters` attribute provides a string of all the letters, lowercase and uppercase, in order. We convert this to a list, and then use list concatenation (the `+`  operator causes two lists to be merged into one) to add one more  character, a space. These are the available characters in our frequency  list (the code would break if we tried to add a letter that wasn't in  the list, but an exception handler could solve this).

The first line inside the function uses a list comprehension to turn the `CHARACTERS`  list into a list of tuples. List comprehensions are an important,  non-object-oriented tool in Python; we'll be covering them in detail in  the next chapter.

Then, we loop over each of the characters in the sentence. We first look up the index of the character in the `CHARACTERS`  list, which we know has the same index in our frequencies list, since  we just created the second list from the first. We then update that  index in the frequencies list by creating a new tuple, discarding the  original one. Aside from garbage collection and memory waste concerns,  this is rather difficult to read!

Like dictionaries, lists are  objects too, and they have several methods that can be invoked upon  them. Here are some common ones:

- The `append(element)` method adds an element to the end of the list
- The `insert(index, element)` method inserts an item at a specific position
- The `count(element)` method tells us how many times an element appears in the list
- The `index()`method tells us the index of an item in the list, raising an exception if it can't find it
- The `find()`method does the same thing, but returns `-1` instead of raising an exception for missing items
- The `reverse()` method does exactly what it says—turns the list around
- The `sort()` method has some rather intricate object-oriented behaviors, which we'll cover now

# Sorting Lists

Without any parameters, `sort` will generally  do as expected. If it's a list of strings, it will place them in  alphabetical order. This operation is case sensitive, so all capital  letters will be sorted before lowercase letters; that is, `Z` comes before `a`.  If it's a list of numbers, they will be sorted in numerical order. If a  list of tuples is provided, the list is sorted by the first element in  each tuple. If a mixture containing unsortable items is supplied, the  sort will raise a `TypeError` exception.

If  we want to place objects we define ourselves into a list and make those  objects sortable, we have to do a bit more work. The special `__lt__ ` method, which stands for **less than**, should be defined on the class to make instances of that class comparable. The `sort` method on the list will access this method on each object to determine where it goes in the list. This method should return `True` if our class is somehow less than the passed parameter, and `False` otherwise. Here's a rather silly class that can be sorted based on either a string or a number:

```python
class WeirdSortee:
    def __init__(self, string, number, sort_num):
        self.string = string
        self.number = number
        self.sort_num = sort_num

    def __lt__(self, object):
        if self.sort_num:
            return self.number < object.number
        return self.string < object.string

    def __repr__(self):
        return f"{self.string}:{self.number}"
```

The `__repr__` method makes it easy to see the two values when we print a list. The `__lt__` method's implementation compares the object to another instance of the same class (or any duck-typed object that has `string`, `number`, and `sort_num` attributes; it will fail if those attributes are missing). The following output illustrates this class in action when it comes to sorting:

```python
>>> a = WeirdSortee('a', 4, True)>>> b = WeirdSortee('b', 3, True)>>> c = WeirdSortee('c', 2, True)>>> d = WeirdSortee('d', 1, True)>>> l = [a,b,c,d]>>> l[a:4, b:3, c:2, d:1]>>> l.sort()>>> l[d:1, c:2, b:3, a:4]>>> for i in l:...     i.sort_num = False...>>> l.sort()>>> l[a:4, b:3, c:2, d:1]
```

The first time we call `sort`, it sorts by numbers because `sort_num` is `True` on all the objects being compared. The second time, it sorts by letters. The `__lt__`  method is the only one we need to implement to enable sorting.  Technically, however, if it is implemented, the class should normally  also implement the similar `__gt__`, `__eq__`, `__ne__`, `__ge__`, and `__le__` methods so that all of the `<`, `>`, `==`, `!=`, `>=`, and `<=` operators also work properly. You can get this for free by implementing `__lt__` and `__eq__`, and then applying the `@total_ordering` class decorator to supply the rest:

```python
from functools import total_ordering 
 
@total_ordering 
class WeirdSortee: 
    def __init__(self, string, number, sort_num): 
        self.string = string 
        self.number = number 
        self.sort_num = sort_num 
 
    def __lt__(self, object): 
        if self.sort_num: 
            return self.number < object.number 
        return self.string < object.string 
 
    def __repr__(self): 
        return f"{self.string}:{self.number}"
 
 def __eq__(self, object): 
        return all(( 
            self.string == object.string, 
            self.number == object.number, 
            self.sort_num == object.number 
        )) 
```

This is useful if we want to be able to use operators on our objects. However, if all we want to do is customize our sort orders, even this is overkill. For such a use case, the `sort` method can take an optional `key`  argument. This argument is a function that can translate each object in  a list into an object that can somehow be compared. For example, we can  use `str.lower` as the key argument to perform a case-insensitive sort on a list of strings, as can be seen in the following:

```python
>>> l = ["hello", "HELP", "Helo"]>>> l.sort()>>> l['HELP', 'Helo', 'hello']>>> l.sort(key=str.lower)>>> l['hello', 'Helo', 'HELP']
```

Remember, even though `lower` is a method on string objects, it is also a function that can accept a single argument, `self`. In other words, `str.lower(item)` is equivalent to `item.lower()`.  When we pass this function as a key, it performs the comparison on  lowercase values instead of doing the default case-sensitive comparison.

There are a few sort key operations  that are so common that the Python team has supplied them so you don't  have to write them yourself. For example, it is common to sort a list of  tuples by something other than the first item in the list. The `operator.itemgetter` method can be used as a key to do this:

```python
>>> from operator import itemgetter>>> l = [('h', 4), ('n', 6), ('o', 5), ('p', 1), ('t', 3), ('y', 2)]>>> l.sort(key=itemgetter(1))>>> l[('p', 1), ('y', 2), ('t', 3), ('h', 4), ('o', 5), ('n', 6)]
```

The `itemgetter` function is the most commonly used one (it works if objects are dictionaries, too), but you will sometimes find use for `attrgetter` and `methodcaller`, which return attributes on an object and the results of method calls on objects for the same purpose. Refer to the `operator` module documentation for more information.