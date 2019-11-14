We discussed briefly in **When Objects Are Alike** how built-in data types can be extended using inheritance. Now, we'll go into more detail as to when we would want to do that.

When  we have a built-in container object that we want to add functionality  to, we have two options. We can either create a new object, which holds  that container as an attribute (composition), or we can subclass the  built-in object and add or adapt methods on it to do what we want  (inheritance).

Composition is usually the best alternative if all  we want to do is use the container to store some objects using that  container's features. That way, it's easy to pass that data structure  into other methods and they will know how to interact with it. But we  need to use inheritance if we want to change the way the container  actually works. For example, if we want to ensure every item in a `list` is a string with exactly five characters, we need to extend `list` and override the `append()` method to raise an exception for invalid input. We'd also minimally have to override `__setitem__(self,``index,``value)`, a special method on lists that is called whenever we use the `x[index]``=``"value"` syntax, and the `extend()` method.

Yes,  lists are objects. All that special non-object-oriented looking syntax  we've been looking at for accessing lists or dictionary keys, looping  over containers, and similar tasks, is actually `syntactic sugar` that  maps to an object-oriented paradigm underneath. We might ask the Python  designers why they did this. Isn't object-oriented programming *always*  better? That question is easy to answer. In the following hypothetical  examples, which is easier to read, as a programmer? Which requires less typing?:

```python
c = a + b 
c = a.add(b) 
 
l[0] = 5 
l.setitem(0, 5) 
d[key] = value 
d.setitem(key, value) 
 
for x in alist: 
    #do something with x 
it = alist.iterator() 
while it.has_next(): 
    x = it.next() 
    #do something with x
```

The  highlighted sections show what object-oriented code might look like (in  practice, these methods actually exist as special double-underscore  methods on associated objects). Python programmers agree that the  non-object-oriented syntax is easier both to read and to write. Yet all  of the preceding Python syntaxes map to object-oriented methods  underneath the hood. These methods have special names (with  double-underscores before and after) to remind us that there is a better  syntax out there. However, it gives us the means to override these  behaviors. For example, we can make a special integer that always returns `0` when we add two of them together, demonstrated as follows:

```python
class SillyInt(int): 
    def __add__(self, num): 
        return 0 
```

This is an extremely bizarre thing to do, granted, but it perfectly illustrates these object-oriented principles in action:

```python
>>> a = SillyInt(1)>>> b = SillyInt(2)>>> a + b0
```

The awesome thing about the `__add__` method is that we can add it to any class we write, and if we use the `+` operator on instances of that class, it will be called. This is how string, tuple, and list concatenation works, for example.

This is true of all the special methods. If we want to use `x``in``myobj` syntax for a custom-defined object, we can implement `__contains__`. If we want to use the `myobj[i]``=``value` syntax, we supply a `__setitem__` method, and if we want to use `something``=``myobj[i]`, we implement `__getitem__`.

There are 33 of these special methods in the `list` class. We can use the `dir` function to see all of them, as follows:

```python
>>> dir(list)['__add__', '__class__', '__contains__', '__delattr__','__delitem__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__rmul__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', 'append', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort'
```

Furthermore, if we desire additional information on how any of these methods work, we can use the `help` function:

```python
>>> help(list.__add__)Help on wrapper_descriptor:__add__(self, value, /)    Return self+value.
```

The `+`  operator on lists concatenates two lists. We don't have room to discuss  all of the available special functions in this book, but you are now  able to explore all this functionality with `dir` and `help`. The [official online Python reference](<https://docs.python.org/3/>) has plenty of useful information as well. Focus especially on the abstract base classes discussed in the `collections` module.

So,  to get back to the earlier point about when we would want to use  composition versus inheritance: if we need to somehow change any of the  methods on the class, including the special methods, we definitely need  to use inheritance. If we used composition, we could write methods  that perform the validation or alterations and ask the caller to use  those methods, but there is nothing stopping them from accessing the  property directly. They could insert an item into our list that does not  have five characters, and that might confuse other methods in the list.

Often,  the need to extend a built-in data type is an indication that we're  using the wrong sort of data type. It is not always the case, but if we  are looking to extend a built-in, we should carefully consider whether  or not a different data structure would be more suitable.