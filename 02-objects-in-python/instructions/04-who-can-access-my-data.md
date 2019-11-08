Most object-oriented programming languages have a concept of **access control**.  This is related to abstraction. Some attributes and methods on an  object are marked private, meaning only that object can access them.  Others are marked protected, meaning only that class and any subclasses  have access. The rest are public, meaning any other object is allowed to  access them.

Python doesn't do this. Python doesn't really  believe in enforcing laws that might someday get in your way. Instead,  it provides unenforced guidelines and best practices. Technically, all  methods and attributes on a class are publicly available. If we want to  suggest that a method should not be used publicly, we can put a note in  docstrings indicating that the method is meant for internal use only  (preferably, with an explanation of how the public-facing API works!).

By convention, we should also prefix an internal attribute or method with an underscore character, `_`. Python programmers will interpret this as **this is an internal variable, think three times before accessing it directly**.  But there is nothing inside the interpreter to stop them from accessing  it if they think it is in their best interest to do so. Because, if  they think so, why should we stop them? We may not have any idea what  future uses our classes may be put to.

There's another thing you  can do to strongly suggest that outside objects don't access a property  or method: prefix it with a double underscore, `__`. This will perform **name mangling**  on the attribute in question. In essence, name mangling means that the  method can still be called by outside objects if they really want to do  so, but it requires extra work and is a strong indicator that you demand that your attribute remains **private**. Here is an example code snippet:

```python
class SecretString:
    """A not-at-all secure way to store a secret string."""

    def __init__(self, plain_string, pass_phrase):
        self.__plain_string = plain_string
        self.__pass_phrase = pass_phrase

    def decrypt(self, pass_phrase):
        """Only show the string if the pass_phrase is correct."""
 if pass_phrase == self.__pass_phrase:
 return self.__plain_string
        else:
            return ""
```

If we load this class and  test it in the interactive interpreter, we can see that it hides the  plain text string from the outside world:

```python
>>> secret_string = SecretString("ACME: Top Secret", "antwerp")>>> print(secret_string.decrypt("antwerp"))ACME: Top Secret>>> print(secret_string.__plain_string)Traceback (most recent call last):  File "<stdin>", line 1, in <module>AttributeError: 'SecretString' object has no attribute'__plain_string'
```

It looks like it works; nobody can access our `plain_string`  attribute without the passphrase, so it must be safe. Before we get too  excited, though, let's see how easy it can be to hack our security:

```python
>>> print(secret_string._SecretString__plain_string)ACME: Top Secret
```

Oh no! Somebody has discovered our secret string. Good thing we checked.

This is Python name mangling at work. When we use a double underscore, the property is prefixed with `_<classname>`. When methods in the class internally  access the variable, they are automatically unmangled. When external  classes wish to access it, they have to do the name mangling themselves.  So, name mangling does not guarantee privacy; it only strongly  recommends it. Most Python programmers will not touch a double  underscore variable on another object unless they have an extremely  compelling reason to do so.

However, most Python programmers will  not touch a single underscore variable without a compelling reason  either. Therefore, there are very few good reasons to use a name-mangled  variable in Python, and doing so can cause grief. For example, a  name-mangled variable may be useful to an as-yet-unknown subclass, and  it would have to do the mangling itself. Let other objects access your  hidden information if they want to. Just let them know, using a  single-underscore prefix or some clear docstrings, that you think this  is not a good idea.