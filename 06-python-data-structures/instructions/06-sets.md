Lists are extremely versatile tools  that suit many container object applications. But they are not useful  when we want to ensure that objects in list are unique. For example, a  song library may contain many songs by the same artist. If we want to  sort through the library and create a list of all the artists, we would  have to check the list to see whether we've added the artist already,  before we add them again.

This is where sets come in. Sets come  from mathematics, where they represent an unordered group of (usually)  unique numbers. We can add a number to a set five times, but it will  show up in the set only once.

In Python, sets can hold any  hashable object, not just numbers. Hashable objects are the same objects  that can be used as keys in dictionaries; so again, lists and  dictionaries are out. Like mathematical sets, they can store only one  copy of each object. So if we're trying to create a list of song  artists, we can create a set of string names and simply add them to the  set. This example starts with a list of (song, artist) tuples and  creates a set of the artists:

```python
song_library = [
    ("Phantom Of The Opera", "Sarah Brightman"),
    ("Knocking On Heaven's Door", "Guns N' Roses"),
    ("Captain Nemo", "Sarah Brightman"),
    ("Patterns In The Ivy", "Opeth"),
    ("November Rain", "Guns N' Roses"),
    ("Beautiful", "Sarah Brightman"),
    ("Mal's Song", "Vixy and Tony"),
]

artists = set()
for song, artist in song_library:
    artists.add(artist)

print(artists)
```

There is no built-in syntax for an empty set as there is for lists and dictionaries; we create a set using the `set()`  constructor. However, we can use the curly braces (borrowed from  dictionary syntax) to create a set, so long as the set contains values.  If we use colons to separate pairs of values, it's a dictionary, as in `{'key': 'value', 'key2': 'value2'}`. If we just separate values with commas, it's a set, as in `{'value', 'value2'}`.

Items can be added individually to the set using its `add` method. If we run this script, we see that the set works as advertised:

```python
{'Sarah Brightman', "Guns N' Roses", 'Vixy and Tony', 'Opeth'}
```

If  you're paying attention to the output, you'll notice that the items are  not printed in the order they were added to the sets. Sets are  inherently unordered due to a hash-based data structure for efficiency.  Because of this lack of ordering, sets cannot have items looked up by  index. The primary purpose of a set is to divide the world into two  groups: **things that are in the set**, and **things that are not in the set**.  It is easy to check whether an item is in a set or to loop over the  items in a set, but if we want to sort or order them, we have to convert  the set to a list. This output shows all three of these activities:

```python
>>> "Opeth" in artistsTrue>>> for artist in artists:...     print("{} plays good music".format(artist))...Sarah Brightman plays good musicGuns N' Roses plays good musicVixy and Tony play good musicOpeth plays good music>>> alphabetical = list(artists)>>> alphabetical.sort()>>> alphabetical["Guns N' Roses", 'Opeth', 'Sarah Brightman', 'Vixy and Tony']
```

While the primary **feature** of a set is uniqueness, that is not its primary **purpose**. Sets are most useful when two or more of them are used in combination. Most of the methods  on the set type operate on other sets, allowing us to efficiently  combine or compare the items in two or more sets. These methods have  strange names, since they use the terminology used in mathematics. We'll  start with three methods that return the same result, regardless of  which is the calling set and which is the called set.

The `union`  method is the most common and easiest to understand. It takes a second  set as a parameter and returns a new set that contains all elements that  are in **either** of the two sets;  if an element is in both original sets, it will, of course, only show up  once in the new set. Union is like a logical `or` operation. Indeed, the `|` operator can be used on two sets to perform the union operation, if you don't like calling methods.

Conversely, the intersection method accepts a second set and returns a new set that contains only those elements that are in **both** sets. It is like a logical `and` operation, and can also be referenced using the `&` operator.

Finally, the `symmetric_difference`  method tells us what's left; it is the set of objects that are in one  set or the other, but not both. The following example illustrates these  methods by comparing some artists preferred by two different people:

```python
first_artists = {
    "Sarah Brightman",
    "Guns N' Roses",
    "Opeth",
    "Vixy and Tony",
}

second_artists = {"Nickelback", "Guns N' Roses", "Savage Garden"}

print("All: {}".format(first_artists.union(second_artists)))
print("Both: {}".format(second_artists.intersection(first_artists)))
print(
    "Either but not both: {}".format(
        first_artists.symmetric_difference(second_artists)
    )
)
```

If we run this code, we see that these three methods do what the print statements suggest they will do:

```python
All: {'Sarah Brightman', "Guns N' Roses", 'Vixy and Tony','Savage Garden', 'Opeth', 'Nickelback'}Both: {"Guns N' Roses"}Either but not both: {'Savage Garden', 'Opeth', 'Nickelback','Sarah Brightman', 'Vixy and Tony'}
```

These methods all return the same result, regardless of which set calls the other. We can say `first_artists.union(second_artists)` or `second_artists.union(first_artists)`  and get the same result. There are also methods that return different  results depending on who is the caller and who is the argument.

These methods include `issubset` and `issuperset`, which are the inverse of each other. Both return a `bool`. The `issubset` method returns `True`, if all of the items in the calling set are also in the set passed as an argument. The `issuperset` method returns `True` if all of the items in the argument are also in the calling set. Thus, `s.issubset(t)` and `t.issuperset(s)` are identical. They will both return `True` if `t` contains all the elements in `s`.

Finally, the `difference` method returns all the elements that are in the calling set, but not in the set passed as an argument; this is like half a `symmetric_difference`. The `difference` method can also be represented by the `-` operator. The following code illustrates these methods in action:

```python
first_artists = {"Sarah Brightman", "Guns N' Roses", 
        "Opeth", "Vixy and Tony"} 
 
bands = {"Guns N' Roses", "Opeth"} 
 
print("first_artists is to bands:") 
print("issuperset: {}".format(first_artists.issuperset(bands))) 
print("issubset: {}".format(first_artists.issubset(bands))) 
print("difference: {}".format(first_artists.difference(bands))) 
print("*"*20) 
print("bands is to first_artists:") 
print("issuperset: {}".format(bands.issuperset(first_artists))) 
print("issubset: {}".format(bands.issubset(first_artists))) 
print("difference: {}".format(bands.difference(first_artists))) 
```

This code simply prints out the response of each method when called from one set on the other. Running it gives us the following output:

```python
first_artists is to bands:issuperset: Trueissubset: Falsedifference: {'Sarah Brightman', 'Vixy and Tony'}********************bands is to first_artists:issuperset: Falseissubset: Truedifference: set()
```

The `difference` method, in the second case, returns an empty set, since there are no items in `bands` that are not in `first_artists`.

The `union`, `intersection`, and `difference`  methods can all take multiple sets as arguments; they will return, as  we might expect, the set that is created when the operation is called on  all the parameters.

So, the methods on sets clearly suggest that  sets are meant to operate on other sets, and that they are not just  containers. If we have data coming in from two different sources and  need to quickly combine them in some way, so as to determine where the  data overlaps or is different, we can use set operations to efficiently  compare them. Or, if we have data incoming that may contain duplicates  of data that has already been processed, we can use sets to compare the  two and process only the new data.

Finally, it is valuable to know that sets are much more efficient than lists when checking for membership using the `in` keyword. If you use the `value in container`syntax on a set or a list, it will return `True` if one of the elements in `container` is equal to `value`, and `False`  otherwise. However, in a list, it will look at every object in the  container until it finds the value, whereas in a set, it simply hashes  the value and checks for membership. This means that a set will find the  value in the same amount of time no matter how big the container is,  but a list will take longer and longer to search for a value as the list  contains more and more values.