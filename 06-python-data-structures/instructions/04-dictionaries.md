Dictionaries are incredibly useful containers  that allow us to map objects directly to other objects. An empty object  with attributes to it is a sort of dictionary; the names of the  properties map to the property values. This is actually closer to the  truth than it sounds; internally, objects normally represent attributes  as a dictionary, where the values are properties or methods on the  objects (see the `__dict__` attribute if you don't believe me). Even the attributes on a module are stored, internally, in a dictionary.

Dictionaries  are extremely efficient at looking up a value, given a specific key  object that maps to that value. They should always be used when you want  to find one object based on some other object. The object that is being  stored is called the **value**; the object that is being used as an index is called the **key**. We've already seen dictionary syntax in some of our previous examples.

Dictionaries can be created either using the `dict()` constructor or using the `{}`  syntax shortcut. In practice, the latter format is almost always used.  We can prepopulate a dictionary by separating the keys from the values  using a colon, and separating the key value pairs using a comma.

For  example, in a stock application, we would most often want to look up  prices by the stock symbol. We can create a dictionary that uses stock  symbols as keys, and tuples (you could also used named tuples or  dataclasses as values, of course) of current, high, and low as values,  like this:

```python
stocks = {
    "GOOG": (1235.20, 1242.54, 1231.06),
    "MSFT": (110.41, 110.45, 109.84),
}
```

As we've seen in previous  examples, we can then look up values in the dictionary by requesting a  key inside square brackets. If the key is not in the dictionary, it will  raise an exception, demonstrated as follows:

```python
>>> stocks["GOOG"](1235.20, 1242.54, 1231.06)>>> stocks["RIM"]Traceback (most recent call last):  File "<stdin>", line 1, in <module>KeyError: 'RIM'
```

We can, of course, catch the `KeyError`  and handle it. But we have other options. Remember, dictionaries are  objects, even if their primary purpose is to hold other objects. As  such, they have several behaviors associated with them. One of the most  useful of these methods is the `get` method; it accepts a key as the first parameter and an optional default value if the key doesn't exist:

```python
>>> print(stocks.get("RIM"))None>>> stocks.get("RIM", "NOT FOUND")'NOT FOUND'
```

For even more control, we can use the `setdefault` method. If the key is in the dictionary, this method behaves just like `get`;  it returns the value for that key. Otherwise, if the key is not in the  dictionary, it will not only return the default value we supply in the  method call (just like `get` does); it will also set the key to that same value. Another way to think of it is that `setdefault` sets a value in the dictionary  only if that value has not previously been set. Then, it returns the  value in the dictionary; either the one that was already there, or the  newly provided default value, as can be seen in the following:

```python
>>> stocks.setdefault("GOOG", "INVALID")(613.3, 625.86, 610.5)>>> stocks.setdefault("BBRY", (10.87, 10.76, 10.90))(10.50, 10.62, 10.39)>>> stocks["BBRY"](10.50, 10.62, 10.39)
```

The `GOOG` stock was already in the dictionary, so when we tried to `setdefault` it to an invalid value, it just returned the value already in the dictionary. `BBRY` was not in the dictionary, so `setdefault`  returned the default value and set the new value in the dictionary for  us. We then check that the new stock is, indeed, in the dictionary.

Three other very useful dictionary methods are `keys()`, `values()`, and `items()`. The first two return an iterator over all the keys and all the values in the dictionary. We can use these like lists or in `for` loops if we want to process all the keys or values. The `items()` method is probably the most useful; it returns an iterator over tuples of `(key, value)` pairs for every item in the dictionary. This works great with tuple unpacking in a `for` loop to loop over associated keys and values. This example does just that to print each stock in the dictionary with its current value:

```python
>>> for stock, values in stocks.items():
... print(f"{stock} last value is {values[0]}")
...
GOOG last value is 1235.2
MSFT last value is 110.41
BBRY last value is 10.5
```

Each key/value tuple is unpacked into two variables named `stock` and `values` (we could use any variable names we wanted, but these both seem appropriate) and then printed in a formatted string.

info> Notice  that the stocks show up in the same order in which they were inserted.  This was not true until Python 3.6, and was not a formal part of the  language definition until Python 3.7. Before that, the underlying dict  implementation used a different underlying data structure that was not  ordered. It's quite rare to need ordering in dictionaries, but if you do  and you need to support Python 3.5 or older, make sure you use the `OrderedDict` class instead, which is available from the `collections` module.

So, there are numerous ways to retrieve data from a dictionary once it has been instantiated: we can use square brackets as index syntax, the `get` method, the `setdefault` method, or iterate over the `items` method, among others.

Finally,  as you likely already know, we can set a value in a dictionary using  the same indexing syntax we use to retrieve a value:

```python
>>> stocks["GOOG"] = (1245.21, 1252.64, 1245.18)>>> stocks['GOOG'](1245.21, 1252.64, 1245.18)
```

Google's  price is higher today, so I've updated the tuple value in the  dictionary. We can use this index syntax to set a value for any key,  regardless of whether the key is in the dictionary. If it is in the  dictionary, the old value will be replaced with the new one; otherwise, a  new key/value pair will be created.

We've been using strings as  dictionary keys, so far, but we aren't limited to string keys. It is  common to use strings as keys, especially when we're storing data in a  dictionary to gather it together (instead of using an object or  dataclass with named properties). But we can also use tuples, numbers,  or even objects we've defined ourselves as dictionary keys. We can even  use different types of keys in a single dictionary, as demonstrated in  the following:

```python
random_keys = {} 
random_keys["astring"] = "somestring" 
random_keys[5] = "aninteger" 
random_keys[25.2] = "floats work too" 
random_keys[("abc", 123)] = "so do tuples" 
 
class AnObject: 
    def __init__(self, avalue): 
        self.avalue = avalue 
 
my_object = AnObject(14) 
random_keys[my_object] = "We can even store objects" 
my_object.avalue = 12 
try: 
    random_keys[[1,2,3]] = "we can't store lists though" 
except: 
    print("unable to store list\n") 
 
for key, value in random_keys.items(): 
    print("{} has value {}".format(key, value)) 
```

This  code shows several different types of keys we can supply to a  dictionary. It also shows one type of object that cannot be used. We've  already used  lists extensively, and we'll be seeing many more details of them in the  next section. Because lists can change at any time (by adding or  removing items, for example), they cannot **hash** to a specific value.

Objects that are **hashable**  basically have a defined algorithm that converts the object into a  unique integer value for rapid lookup in the dictionary. This hash is  what is actually used to find values in a dictionary. For example,  strings map to integers based on the byte values of the characters in  the string, while tuples combine hashes of the items inside the tuple.  Any two objects that are somehow considered equal (such as strings with  the same characters or tuples with the same values) should have the same  hash value, and the hash value for an object should never ever change.  Lists, however, can have their contents changed, which would change  their hash value (two lists should only be equal if their contents are  the same). Because of this, they can't be used as dictionary keys. For  the same reason, dictionaries cannot be used as keys into other  dictionaries.

In contrast, there are no limits on the types of  objects that can be used as dictionary values. We can use a string key  that maps to a list value, for example, or we can have a nested dictionary as a value in another dictionary.

# Dictionary Use Cases

Dictionaries  are extremely versatile and have numerous uses. There are two major  ways that dictionaries can be used. The first is dictionaries where all  the keys represent different instances of similar objects; for example,  our stock dictionary. This is an indexing system. We use the stock  symbol as an index to the values. The values could even have been  complicated, self-defined objects that had methods to make buy and sell decisions or set a stop-loss, rather than our simple tuples.

The  second design is dictionaries where each key represents some aspect of a  single structure; in this case, we'd probably use a separate dictionary  for each object, and they'd all have similar (though often not  identical) sets of keys. This latter situation can often also be solved  with named tuples or dataclasses. This can be confusing; how do we  decide which to use?

We should typically use dataclasses when we  know exactly what attributes the data must store, especially if we also  want to use the class definition as documentation for the end user.

Dataclasses  are a newer addition to the Python standard library (since Python 3.7).  I expect them to replace named tuples for a huge number of use cases.  Named tuples may also be useful if you are going to be returning them  from functions. That allows the calling function to use tuple unpacking  if it is useful to do so. Dataclasses are not iterable, so you can't  loop over or unpack their values.

On the other hand, dictionaries  would be a better choice if the keys describing the object are not known  in advance, or if different objects will have some variety in their  keys. If we don't know in advance what all the keys are going to be,  it's probably better to use a dictionary.

info> Technically,  most Python objects are implemented using dictionaries under the hood.  You can see this by loading an object into the interactive interpreter  and looking at the `obj.__dict__` magic attribute. When you access an attribute on an object using `obj.attr_name`, it essentially translates the lookup to `obj['attr_name']` under the hood. It's more complicated than that, but you get the gist. Even dataclasses have a `__dict__`  attribute, which just goes to show how versatile dictionaries really  are. Note that not all objects are stored in dictionaries, however.  There are a few special types, such as lists, dictionaries, and  datetimes that are implemented in a different way, mostly for efficiency  purposes. It would certainly be odd if an instance of `dict` had a `__dict__` attribute that was an instance of `dict`, wouldn't it?

# Using defaultdict

We've seen how to use `setdefault` to set a default value if a key doesn't exist, but this can get a bit monotonous if we need to set a default value every  time we look up a value. For example, if we're writing code that counts  the number of times a letter occurs in a given sentence, we could do  the following:

```python
def letter_frequency(sentence): 
    frequencies = {} 
    for letter in sentence: 
        frequency = frequencies.setdefault(letter, 0) 
        frequencies[letter] = frequency + 1 
    return frequencies 
```

Every time we access the  dictionary, we need to check that it has a value already, and if not,  set it to zero. When something like this needs to be done every time an  empty key is requested, we can use a different version of the  dictionary, called `defaultdict`:

```python
from collections import defaultdict 
def letter_frequency(sentence): 
    frequencies = defaultdict(int) 
    for letter in sentence: 
        frequencies[letter] += 1 
    return frequencies 
```

This code looks like it couldn't possibly work. The `defaultdict` accepts a function in its constructor. Whenever a key is accessed that is not already in the dictionary, it calls that function, with no parameters, to create a default value.

In this case, the function it calls is `int`,  which is the constructor for an integer object. Normally, integers are  created simply by typing an integer number into our code, and if we do  create one using the `int` constructor, we  pass it the item we want to create (for example, to convert a string of  digits into an integer). But if we call `int` without any arguments, it returns, conveniently, the number zero. In this code, if the letter doesn't exist in the `defaultdict`,  the number zero is returned when we access it. Then, we add one to this  number to indicate that we've found an instance of that letter, and the  next time we find one, that number will be returned and we can  increment the value again.

The `defaultdict`  is useful for creating dictionaries of containers. If we want to create  a dictionary of closing stock prices for the past 30 days, we could use  a stock symbol as the key and store the prices in `list`; the first time we access the stock price, we would want it to create an empty list. Simply pass `list` into the `defaultdict`,  and it will be called every time an empty key is accessed. We can do  similar things with sets or even empty dictionaries if we want to  associate one with a key.

Of course, we can also write our own functions and pass them into the `defaultdict`. Suppose we want to create a `defaultdict`  where each new element contains a tuple of the number of items inserted  into the dictionary at that time and an empty list to hold other  things. It's unlikely that we would want to create such an object, but let's have a look:

```python
from collections import defaultdict

 
num_items = 0 

def tuple_counter(): 
    global num_items 
    num_items += 1 
    return (num_items, []) 
 
d = defaultdict(tuple_counter) 
```

When we run this code, we can access empty keys and insert them into the list all in a single statement:

```python
>>> d = defaultdict(tuple_counter)>>> d['a'][1].append("hello")>>> d['b'][1].append('world')>>> ddefaultdict(<function tuple_counter at 0x82f2c6c>,{'a': (1, ['hello']), 'b': (2, ['world'])})
```

When we print `dict` at the end, we see that the counter really was working.

info> This example, while succinctly demonstrating how to create our own function for `defaultdict`, is not actually very good code; using a global variable means that if we created four different `defaultdict` segments that each used `tuple_counter`,  it would count the number of entries in all dictionaries, rather than  having a different count for each one. It would be better to create a  class and pass a method on that class to `defaultdict`.

## Counter

You'd think that you couldn't get much simpler than `defaultdict(int)`, but the **I want to count specific instances in an iterable** use  case is common enough that the Python developers created a specific  class for it. The previous code that counts characters in a string can  easily be calculated in a single line:

```python
from collections import Counter 
def letter_frequency(sentence): 
    return Counter(sentence)
```

The `Counter`  object behaves like a beefed-up dictionary where the keys are the items  being counted and the values are the quantities of such items. One of  the most useful functions is the `most_common()` method. It returns a list of (key, count) tuples ordered by the count. You can optionally pass an integer argument into `most_common()` to request only the top most common elements. For example, you could write a simple polling application as follows:

```python
from collections import Counter 
 
responses = [ 
    "vanilla", 
    "chocolate", 
    "vanilla", 
    "vanilla", 
    "caramel", 
    "strawberry", 
    "vanilla" 
] 
 
print( 
    "The children voted for {} ice cream".format( 
 Counter(responses).most_common(1)[0][0] 
    ) 
) 
```

Presumably, you'd get the responses from a database or by using a computer vision algorithm to count the kids who raised their hands. Here, we hardcode it so that we can test the `most_common`  method. It returns a list that has only one element (because we  requested one element in the parameter). This element stores the name of  the top choice at position zero, hence the double `[0][0]`  at the end of the call. I think they look like a surprised face, don't  you? Your computer is probably amazed it can count data so easily. It's  ancestor, Hollerith's tabulating machine developed for the 1890 US  census, must be so jealous!