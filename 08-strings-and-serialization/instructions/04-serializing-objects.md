Nowadays, we take the ability to write data to a file and retrieve it  at an arbitrary later date for granted. As convenient as this is  (imagine the state of computing  if we couldn't store anything!), we often find ourselves converting  data we have stored in a nice object or design pattern in memory into  some kind of clunky text or binary format for storage, transfer over the  network, or remote invocation on a distant server.

The Python `pickle`  module is an object-oriented way to store objects directly in a special  storage format. It essentially converts an object (and all the objects  it holds as attributes) into a sequence of bytes that can be stored or  transported however we see fit.

For basic tasks, the `pickle`  module has an extremely simple interface. It comprises four basic  functions for storing and loading data: two for manipulating file-like  objects, and two for manipulating `bytes` objects (the latter are just shortcuts to the file-like interface, so we don't have to create a `BytesIO` file-like object ourselves).

The `dump` method accepts an object to be written and a file-like object to write the serialized bytes to. This object must have a `write` method (or it wouldn't be file-like), and that method must know how to handle a `bytes` argument (so, a file opened for text output wouldn't work).

The `load`  method does exactly the opposite; it reads a serialized object from a  file-like object. This object must have the proper file-like `read` and `readline` arguments, each of which must, of course, return `bytes`. The `pickle` module will load the object from these bytes and the `load` method will return the fully reconstructed object. Here's an example that stores and then loads some data in a list object:

```python
import pickle 
 
some_data = ["a list", "containing", 5, 
        "values including another list", 
        ["inner", "list"]] 
 
with open("pickled_list", 'wb') as file: 
    pickle.dump(some_data, file) 
 
with open("pickled_list", 'rb') as file: 
    loaded_data = pickle.load(file) 
 
print(loaded_data) 
assert loaded_data == some_data 
```

This code works as  advertised: the objects are stored in the file and then loaded from the  same file. In each case, we open the file using a `with`  statement so that it is automatically closed. The file is first opened  for writing and then a second time for reading, depending on whether we  are storing or loading data.

The `assert`  statement at the end would raise an error if the newly loaded object  was not equal to the original object. Equality does not imply that they  are the same object. Indeed, if we print the `id()`  of both objects, we would discover they are different. However, because  they are both lists whose contents are equal, the two lists are also  considered equal.

The `dumps` and `loads` functions behave much like their file-like counterparts, except they return or accept `bytes` instead of file-like objects. The `dumps` function requires only one argument, the object to be stored, and it returns a serialized `bytes` object. The `loads` function requires a `bytes` object and returns the restored object. The `'s'` character in the method names is short for string; it's a legacy name from ancient versions of Python, where `str` objects were used instead of `bytes`.

It is possible to call `dump` or `load` on a single open file more than once. Each call to `dump` will store a single object (plus any objects it is composed of or contains), while a call to `load` will load and return just one object. So, for a single file, each separate call to `dump` when storing the object should have an associated call to `load` when restoring at a later date.

# Customizing Pickles

With most common Python objects, pickling **just works**. Basic primitives such as integers, floats, and strings can be pickled, as can any container  objects, such as lists or dictionaries, provided the contents of those  containers are also picklable. Further, and importantly, any object can  be pickled, so long as all of its attributes are also picklable.

So,  what makes an attribute unpicklable? Usually, it has something to do  with time-sensitive attributes that it would not make sense to load in  the future. For example, if we have an open network socket, open file,  running thread, or database connection stored as an attribute on an  object, it would not make sense to pickle these objects; a lot of  operating system state would simply be gone when we attempted to reload  them later. We can't just pretend a thread or socket connection exists  and make it appear! No, we need to somehow customize how such transient  data is stored and restored.

Here's a class that loads the contents of a web page every hour to ensure that they stay up to date. It uses the `threading.Timer` class to schedule the next update:

```python
from threading import Timer 
import datetime 
from urllib.request import urlopen 
 
class UpdatedURL: 
    def __init__(self, url): 
        self.url = url 
        self.contents = '' 
        self.last_updated = None 
        self.update() 
 
    def update(self): 
        self.contents = urlopen(self.url).read() 
        self.last_updated = datetime.datetime.now() 
        self.schedule() 
 
    def schedule(self): 
        self.timer = Timer(3600, self.update) 
        self.timer.setDaemon(True) 
        self.timer.start() 
```

`url`, `contents`, and `last_updated` are all pickleable, but if we try to pickle an instance of this class, things go a little nutty on the `self.timer` instance:

```python
>>> u = UpdatedURL("http://dusty.phillips.codes")
^[[Apickle.dumps(u)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can't pickle _thread.lock objects
```

That's not a very useful error, but it looks like we're trying to pickle something we shouldn't be. That would be the `Timer` instance; we're storing a reference to `self.timer` in the schedule method, and that attribute cannot be serialized.

When `pickle` tries to serialize an object, it simply tries to store the object's `__dict__` attribute; `__dict__` is a dictionary mapping all the attribute names on the object to their values. Luckily, before checking `__dict__`, `pickle` checks to see whether a `__getstate__` method exists. If it does, it will store the return value of that method instead of the `__dict__`.

Let's add a `__getstate__` method to our `UpdatedURL` class that simply returns a copy of the `__dict__` without a timer:

```python
    def __getstate__(self): 
        new_state = self.__dict__.copy() 
        if 'timer' in new_state: 
            del new_state['timer'] 
        return new_state 
```

If we pickle the object now, it will no longer fail. And we can even successfully restore that object using `loads`.  However, the restored object doesn't have a timer attribute, so it will  not be refreshing the content like it is designed to do. We need to  somehow create a new timer (to replace the missing one) when the object  is unpickled.

As we might expect, there is a complementary `__setstate__` method that can be implemented to customize unpickling. This method accepts a single argument, which is the object returned by `__getstate__`. If we implement both methods, `__getstate__` is not required to return a dictionary, since `__setstate__` will know what to do with whatever object `__getstate__` chooses to return. In our case, we simply want to restore the `__dict__`, and then create a new timer:

```python
 def __setstate__(self, data): self.__dict__ = data self.schedule() 
```

The `pickle`  module is very flexible and provides other tools to further customize  the pickling process if you need them. However, these are beyond the  scope of this book. The tools we've covered are sufficient for many  basic pickling tasks. Objects to be pickled are normally relatively  simple data objects; we likely would not pickle an entire running  program or complicated design pattern, for example.

# Serializing Web Objects

It is not a good idea to load a pickled object from an unknown  or untrusted source. It is possible to inject arbitrary code into a  pickled file to maliciously attack a computer via the pickle. Another  disadvantage of pickles is that they can only be loaded by other Python  programs, and cannot be easily shared with services written in other  languages.

There are many formats that have been used for this purpose over the years. **Extensible Markup Language** (**XML**) used to be very popular, especially with Java developers. **Yet Another Markup Language** (**YAML**) is another format that you may see referenced occasionally. Tabular data is frequently exchanged in the **Comma-Separated Value** (**CSV**) format. Many of these are fading into obscurity and there are many more that you will encounter over time. Python has solid standard or third-party libraries for all of them.

Before  using such libraries on untrusted data, make sure to investigate  security concerns with each of them. XML and YAML, for example, both  have obscure features that, used maliciously, can allow arbitrary  commands to be executed on the host machine. These features may not be  turned off by default. Do your research.

**JavaScript Object Notation** (**JSON**) is a human-readable format for exchanging primitive data. JSON is a standard format that can be interpreted by a wide  array of heterogeneous client systems. Hence, JSON is extremely useful  for transmitting data between completely decoupled systems. Further,  JSON does not have any support for executable code, only data can be  serialized; thus, it is more difficult to inject malicious statements  into it.

Because JSON can be easily interpreted by JavaScript  engines, it is often used for transmitting data from a web server to a  JavaScript-capable web browser. If the web application serving the data  is written in Python, it needs a way to convert internal data into the  JSON format.

There is a module to do this, predictably named `json`. This module provides a similar interface to the `pickle` module, with `dump`, `load`, `dumps`, and `loads` functions. The default calls to these functions are nearly identical to those in `pickle`,  so let's not repeat the details. There are a couple of differences;  obviously, the output of these calls is valid JSON notation, rather than a pickled object. In addition, the `json` functions operate on `str` objects, rather than `bytes`. Therefore, when dumping to or loading from a file, we need to create text files rather than binary ones.

The JSON serializer is not as robust as the `pickle`  module; it can only serialize basic types such as integers, floats, and  strings, and simple containers such as dictionaries and lists. Each of  these has a direct mapping to a JSON representation, but JSON is unable  to represent classes, methods, or functions. It is not possible to  transmit complete objects in this format. Because the receiver of an  object we have dumped to JSON format is normally not a Python object, it  would not be able to understand classes or methods in the same way that  Python does, anyway. In spite of the O for Object in its name, JSON is a  **data** notation; objects, as you recall, are composed of both data and behaviors.

If we do have objects for which we want to serialize only the data, we can always serialize the object's `__dict__`  attribute. Or we can semi-automate this task by supplying custom code  to create or parse a JSON serializable dictionary from certain types of  objects.

In the `json` module, both the object storing and loading functions accept optional arguments to customize the behavior. The `dump` and `dumps` methods accept a poorly named `cls` (short for class, which is a reserved keyword) keyword argument. If passed, this should be a subclass of the `JSONEncoder` class, with the `default` method overridden. This method accepts an arbitrary object and converts it to a dictionary that `json` can digest. If it doesn't know how to process the object, we should call the `super()` method, so that it can take care of serializing basic types in the normal way.

The `load` and `loads` methods also accept such a `cls` argument that can be a subclass of the inverse class, `JSONDecoder`. However, it is normally sufficient to pass a function into these methods using the `object_hook` keyword argument. This function accepts a dictionary and returns an object; if it doesn't know what to do with the input dictionary, it can return it unmodified.

Let's look at an example. Imagine we have the following simple contact class that we want to serialize:

```python
class Contact: 
    def __init__(self, first, last): 
        self.first = first 
        self.last = last 
 
    @property 
    def full_name(self): 
        return("{} {}".format(self.first, self.last)) 
```

We could just serialize the `__dict__` attribute:

```python
>>> c = Contact("John", "Smith")>>> json.dumps(c.__dict__)'{"last": "Smith", "first": "John"}'
```

But  accessing special (double-underscore) attributes in this fashion is  kind of crude. Also, what if the receiving code (perhaps some JavaScript  on a web page) wanted that `full_name` property to be supplied? Of course, we could construct the dictionary by hand, but let's create a custom encoder instead:

```python
import json


class ContactEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Contact):
            return {
                "is_contact": True,
                "first": obj.first,
                "last": obj.last,
                "full": obj.full_name,
            }
        return super().default(obj)
```

The `default` method basically checks to see what  kind of object we're trying to serialize. If it's a contact, we convert  it to a dictionary manually. Otherwise, we let the parent class handle  serialization (by assuming that it is a basic type, which `json`  knows how to handle). Notice that we pass an extra attribute to  identify this object as a contact, since there would be no way to tell  upon loading it. This is just a convention; for a more generic  serialization mechanism, it might make more sense to store a string type  in the dictionary, or possibly even the full class name, including  package and module. Remember that the format of the dictionary depends  on the code at the receiving end; there has to be an agreement as to how  the data is going to be specified.

We can use this class to encode a contact by passing the class (not an instantiated object) to the `dump` or `dumps` function:

```python
>>> c = Contact("John", "Smith")>>> json.dumps(c, cls=ContactEncoder)'{"is_contact": true, "last": "Smith", "full": "John Smith","first": "John"}'
```

For decoding, we can write a function that accepts a dictionary and checks the existence of the `is_contact` variable to decide whether to convert it to a contact:

```python
def decode_contact(dic):
    if dic.get("is_contact"):
        return Contact(dic["first"], dic["last"])
    else:
        return dic
```

We can pass this function to the `load` or `loads` function using the `object_hook` keyword argument:

```python
>>> data = ('{"is_contact": true, "last": "smith",'     '"full": "john smith", "first": "john"}')>>> c = json.loads(data, object_hook=decode_contact)>>> c<__main__.Contact object at 0xa02918c>>>> c.full_name'john smith'
```