The singleton pattern is one of the most controversial patterns; many have accused it of being an **anti-pattern**,  a pattern that should be avoided, not promoted. In Python, if someone  is using the singleton pattern, they're almost certainly doing something  wrong, probably because they're coming from a more restrictive  programming language.

So, why discuss it at all? Singleton is one  of the most famous of all design patterns. It is useful in overly  object-oriented languages, and is a vital part of traditional  object-oriented programming. More relevantly, the idea behind singleton  is useful, even if we implement the concept in a totally different way  in Python.

The basic idea behind the singleton pattern is to allow  exactly one instance of a certain object to exist. Typically, this  object is a sort of manager class like those we discussed in **When to Use Object-Oriented Programming**.  Such objects often need to be referenced by a wide variety of other  objects, and passing references to the manager object around to the  methods and constructors that need them can make code hard to read.

Instead,  when a singleton is used, the separate objects request the single  instance of the manager object from the class, so a reference to it need  not to be passed around. The UML diagram doesn't fully describe it, but  here it is for completeness:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/cbb45eaa-5486-4f7d-b292-fd75d4ffdf80.png)

In most programming environments, singletons are enforced by making  the constructor private (so no one can create additional instances of  it), and then providing a static method to retrieve the single instance.  This method creates a new instance the first time it is called, and  then returns that same instance for all subsequent calls.

# Singleton Implementation

Python doesn't have private constructors, but for this purpose, we can use the `__new__` class method to ensure that only one instance is ever created:

```python
class OneOnly: 
    _singleton = None 
    def __new__(cls, *args, **kwargs): 
        if not cls._singleton: 
            cls._singleton = super(OneOnly, cls 
                ).__new__(cls, *args, **kwargs) 
        return cls._singleton 
```

When `__new__`  is called, it normally constructs a new instance of that class. When we  override it, we first check whether our singleton instance has been  created; if not, we create it using a `super` call. Thus, whenever we call the constructor on `OneOnly`, we always get the exact same instance:

```python
>>> o1 = OneOnly()>>> o2 = OneOnly()>>> o1 == o2True>>> o1<__main__.OneOnly object at 0xb71c008c>>>> o2<__main__.OneOnly object at 0xb71c008c>
```

The  two objects are equal and located at the same address; thus, they are  the same object. This particular implementation isn't very transparent,  since it's not obvious that a singleton object has been created.  Whenever we call a constructor, we expect a new instance of that object;  in this case, that contract is violated. Perhaps, good docstrings on  the class could alleviate this problem if we really think we need a  singleton.

But we don't need it. Python coders frown on forcing  the users of their code into a specific mindset. We may think only one  instance of a class will ever be required, but other programmers may  have different ideas. Singletons can interfere with distributed  computing, parallel programming, and automated testing, for example. In  all those cases, it can be very useful to have multiple or alternative  instances of a specific object, even though a *normal* operation may never require one.

# Module Variables can Mimic Singletons

Normally, in Python, the singleton pattern can be sufficiently mimicked using module-level variables. It's not as **safe** as a singleton in that people could reassign those variables at any time, but as with the private variables we discussed in **Objects in Python**,  this is acceptable in Python. If someone has a valid reason to change  those variables, why should we stop them? It also doesn't stop people  from instantiating multiple instances of the object, but again, if they  have a valid reason to do so, why interfere?

Ideally, we should give them a mechanism to get access to the **default singleton** value,  while also allowing them to create other instances if they need them.  While technically not a singleton at all, it provides the most Pythonic  mechanism for singleton-like behavior.

To use module-level  variables instead of a singleton, we instantiate an instance of the  class after we've defined it. We can improve our state pattern to use  singletons. Instead of creating a new object every time we change  states, we can create a module-level variable that is always accessible:

```python
class Node:
    def __init__(self, tag_name, parent=None):
        self.parent = parent
        self.tag_name = tag_name
        self.children = []
        self.text = ""

    def __str__(self):
        if self.text:
            return self.tag_name + ": " + self.text
        else:
            return self.tag_name


class FirstTag:
    def process(self, remaining_string, parser):
        i_start_tag = remaining_string.find("<")
        i_end_tag = remaining_string.find(">")
        tag_name = remaining_string[i_start_tag + 1 : i_end_tag]
        root = Node(tag_name)
        parser.root = parser.current_node = root
        parser.state = child_node
        return remaining_string[i_end_tag + 1 :]


class ChildNode:
    def process(self, remaining_string, parser):
        stripped = remaining_string.strip()
        if stripped.startswith("</"):
            parser.state = close_tag
        elif stripped.startswith("<"):
            parser.state = open_tag
        else:
            parser.state = text_node
        return stripped


class OpenTag:
    def process(self, remaining_string, parser):
        i_start_tag = remaining_string.find("<")
        i_end_tag = remaining_string.find(">")
        tag_name = remaining_string[i_start_tag + 1 : i_end_tag]
        node = Node(tag_name, parser.current_node)
        parser.current_node.children.append(node)
        parser.current_node = node
        parser.state = child_node
        return remaining_string[i_end_tag + 1 :]


class TextNode:
    def process(self, remaining_string, parser):
        i_start_tag = remaining_string.find("<")
        text = remaining_string[:i_start_tag]
        parser.current_node.text = text
        parser.state = child_node
        return remaining_string[i_start_tag:]


class CloseTag:
    def process(self, remaining_string, parser):
        i_start_tag = remaining_string.find("<")
        i_end_tag = remaining_string.find(">")
        assert remaining_string[i_start_tag + 1] == "/"
        tag_name = remaining_string[i_start_tag + 2 : i_end_tag]
        assert tag_name == parser.current_node.tag_name
        parser.current_node = parser.current_node.parent
        parser.state = child_node
        return remaining_string[i_end_tag + 1 :].strip()


first_tag = FirstTag()
child_node = ChildNode()
text_node = TextNode()
open_tag = OpenTag()
close_tag = CloseTag()
```

All  we've done is create instances of the various state classes that can be  reused. Notice how we can access these module variables inside the  classes, even before the variables have been defined? This is because  the code inside the classes is not executed until the method is called,  and by this point, the entire module will have been defined.

The  difference in this example is that instead of wasting memory creating a  bunch of new instances that must be garbage collected, we are reusing a  single state object for each state. Even if multiple parsers are running  at once, only these state classes need to be used.

When we originally created the state-based parser, you may have wondered why we didn't pass the parser object to `__init__` on each individual state, instead of passing it into the `process` method as we did. The state could then have been referenced as `self.parser`. This is a perfectly valid implementation  of the state pattern, but it would not have allowed leveraging the  singleton pattern. If the state objects maintain a reference to the  parser, then they cannot be used simultaneously to reference other  parsers.

info> Remember,  these are two different patterns with different purposes; the fact that  singleton's purpose may be useful for implementing the state pattern  does not mean the two patterns are related.