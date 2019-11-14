Let's start with the most basic Python built-in, one that we've seen many times already, the one that we've extended in every class we have created: the `object`. Technically, we can instantiate an `object` without writing a subclass, as follows:

```python
>>> o = object()>>> o.x = 5Traceback (most recent call last):  File "<stdin>", line 1, in <module>AttributeError: 'object' object has no attribute 'x'
```

Unfortunately, as you can see, it's not possible to set any attributes on an `object`  that was instantiated directly. This isn't because the Python  developers wanted to force us to write our own classes, or anything so  sinister. They did this to save memory; a lot of memory. When Python  allows an object to have arbitrary attributes, it takes a certain amount  of system memory to keep track of what attributes each object has, for  storing both the attribute name and its value. Even if no attributes are  stored, memory is allocated for **potential**  new attributes. Given the dozens, hundreds, or thousands of objects  (every class extends an object) in a typical Python program; this small  amount of memory would quickly become a large amount of memory. So,  Python disables arbitrary properties on `object`, and several other built-ins, by default.

info> It is possible to restrict arbitrary properties on our own classes using **slots**.  Slots are beyond the scope of this book, but you now have a search term  if you are looking for more information. In normal use, there isn't  much benefit to using slots, but if you're writing an object that will  be duplicated thousands of times throughout the system, they can help  save memory, just as they do for `object`.

It is, however, trivial to create an empty object class of our own; we saw it in our earliest example:

```python
class MyObject: 
    pass 
```

And, as we've already seen, it's possible to set attributes on such classes as follows:

```python
>>> m = MyObject()>>> m.x = "hello">>> m.x'hello'
```

If  we wanted to group properties together, we could store them in an empty  object like this. But we are usually better off using other built-ins  designed for storing data. It has been stressed throughout this book  that classes and objects should only be used when you want to specify **both**  data and behaviors. The main reason to write an empty class is to  quickly block something out, knowing we'll come back later to add  behavior. It is much easier to adapt behaviors to a class than it is to  replace a data structure with an object and change all references to it.  Therefore, it is important to decide from the outset whether the data  is just data, or whether it is an object in disguise. Once that design  decision is made, the rest of the design naturally falls into place.