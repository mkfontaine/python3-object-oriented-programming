Technically, every class we create uses inheritance. All Python classes are subclasses of the special built-in class named `object`. This class provides very little  in terms of data and behaviors (the behaviors it does provide are all  double-underscore methods intended for internal use only), but it does  allow Python to treat all objects in the same way.

If we don't explicitly inherit from a different class, our classes will automatically inherit from `object`. However, we can openly state that our class derives from `object` using the following syntax:

```python
class MySubClass(object): 
    pass 
```

This is inheritance! This example is, technically, no different from our very first example in [**Objects in Python**, since Python 3 automatically inherits from `object` if we don't explicitly provide a different **superclass**.  A superclass, or parent class, is a class that is being inherited from.  A subclass is a class that is inheriting from a superclass. In this  case, the superclass is `object`, and `MySubClass` is the subclass. A subclass is also said to be derived from its parent class or that the subclass extends the parent.

As  you've probably figured out from the example, inheritance requires a  minimal amount of extra syntax over a basic class definition. Simply  include the name of the parent class inside parentheses between the  class name and the colon that follows. This is all we have to do to tell  Python that the new class should be derived from the given superclass.

How  do we apply inheritance in practice? The simplest and most obvious use  of inheritance is to add functionality to an existing class. Let's start  with a simple contact manager that tracks the name and email address of  several people. The `Contact` class is  responsible for maintaining a list of all contacts in a class variable,  and for initializing the name and address for an individual contact:

```python
class Contact:
    all_contacts = []

    def __init__(self, name, email):
        self.name = name
        self.email = email
        Contact.all_contacts.append(self)
```

This example introduces us to **class variables**. The `all_contacts` list, because it is part of the class definition, is shared by all instances of this class. This means that there is only one `Contact.all_contacts` list. We can also access it as `self.all_contacts` from within any method on an instance of the `Contact` class. If a field can't be found on the object (via `self`), then it will be found on the class and will thus refer to the same single list.

info> Be careful with this syntax, for if you ever *set* the variable using `self.all_contacts`, you will actually be creating a **new** instance variable associated just with that object. The class variable will still be unchanged and accessible as `Contact.all_contacts`.

This  is a simple class that allows us to track a couple of pieces of data  about each contact. But what if some of our contacts are also suppliers  that we need to order supplies from? We could add an `order` method to the `Contact`  class, but that would allow people to accidentally order things from  contacts who are customers or family friends. Instead, let's create a  new `Supplier` class that acts like our `Contact` class, but has an additional `order` method:

```python
class Supplier(Contact):
    def order(self, order):
        print(
            "If this were a real system we would send "
            f"'{order}' order to '{self.name}'"
        )
```

Now, if we test this class in our trusty  interpreter, we see that all contacts, including suppliers, accept a  name and email address in their `__init__`, but that only suppliers have a functional order method:

```python
>>> c = Contact("Some Body", "somebody@example.net")>>> s = Supplier("Sup Plier", "supplier@example.net")>>> print(c.name, c.email, s.name, s.email)Some Body somebody@example.net Sup Plier supplier@example.net>>> c.all_contacts[<__main__.Contact object at 0xb7375ecc>, <__main__.Supplier object at 0xb7375f8c>]>>> c.order("I need pliers")Traceback (most recent call last):  File "<stdin>", line 1, in <module>AttributeError: 'Contact' object has no attribute 'order'>>> s.order("I need pliers")If this were a real system we would send 'I need pliers' order to'Sup Plier '
```

So, now our `Supplier` class can do everything a contact can do (including adding itself to the list of `all_contacts`) and all the special things it needs to handle as a supplier. This is the beauty of inheritance.

# Extending Built-Ins

One interesting use of this kind of inheritance is adding functionality to built-in classes. In the `Contact`  class seen earlier, we are adding contacts to a list of all contacts.  What if we also wanted to search that list by name? Well, we could add a  method on the `Contact` class to search it, but it feels like this method actually belongs to the list itself. We can do this using inheritance:

```python
class ContactList(list):
    def search(self, name):
        """Return all contacts that contain the search value
        in their name."""
        matching_contacts = []
        for contact in self:
            if name in contact.name:
                matching_contacts.append(contact)
        return matching_contacts


class Contact:
    all_contacts = ContactList()

    def __init__(self, name, email):
        self.name = name
        self.email = email
        Contact.all_contacts.append(self)
```

Instead of instantiating a normal list as our class variable, we create a new `ContactList` class that extends the built-in `list` data type. Then, we instantiate this subclass as our `all_contacts` list. We can test the new search functionality as follows:

```python
>>> c1 = Contact("John A", "johna@example.net")>>> c2 = Contact("John B", "johnb@example.net")>>> c3 = Contact("Jenna C", "jennac@example.net")>>> [c.name for c in Contact.all_contacts.search('John')]['John A', 'John B']
```

Are you wondering how we changed the built-in syntax `[]` into something we can inherit from? Creating an empty list with `[]` is actually a shortcut for creating an empty list using `list()`; the two syntaxes behave identically:

```python
>>> [] == list()True
```

In reality, the `[]` syntax is actually so-called **syntactic sugar** that calls the `list()` constructor under the hood. The `list` data type is a class that we can extend. In fact, the list itself extends the `object` class:

```python
>>> isinstance([], object)True
```

As a second example, we can extend the `dict` class, which is, similar to the list, the class that is constructed when using the `{}` syntax shorthand:

```python
class LongNameDict(dict): 
    def longest_key(self): 
        longest = None 
        for key in self: 
            if not longest or len(key) > len(longest): 
                longest = key 
        return longest 
```

This is easy to test in the interactive interpreter:

```python
>>> longkeys = LongNameDict()>>> longkeys['hello'] = 1>>> longkeys['longest yet'] = 5>>> longkeys['hello2'] = 'world'>>> longkeys.longest_key()'longest yet'
```

Most built-in types can be similarly extended. Commonly extended built-ins are `object`, `list`, `set`, `dict`, `file`, and `str`. Numerical types such as `int` and `float` are also occasionally inherited from.

# Overriding and super

So, inheritance is great for **adding** new behavior to existing classes, but what about **changing** behavior? Our `Contact` class allows only a name and an email address. This may be sufficient for most contacts, but what if we want to add a phone number for our close friends?

As we saw in **Objects in Python**, we can do this easily by just setting a `phone`  attribute on the contact after it is constructed. But if we want to  make this third variable available on initialization, we have to  override `__init__`. Overriding means  altering or replacing a method of the superclass with a new method (with  the same name) in the subclass. No special syntax is needed to do this;  the subclass's newly created method is automatically called instead of  the superclass's method. As shown in the following code:

```python
class Friend(Contact): 
 def __init__(self, name, email, phone):         self.name = name 
        self.email = email 
        self.phone = phone 
```

Any method can be overridden, not just `__init__`. Before we go on, however, we need to address some problems in this example. Our `Contact` and `Friend` classes have duplicate code to set up the `name` and `email`  properties; this can make code maintenance complicated, as we have to  update the code in two or more places. More alarmingly, our `Friend` class is neglecting to add itself to the `all_contacts` list we have created on the `Contact` class.

What we really need is a way to execute the original `__init__` method on the `Contact` class from inside our new class. This is what the `super` function does; it returns the object as an instance of the parent class, allowing us to call the parent method directly:

```python
class Friend(Contact): 
    def __init__(self, name, email, phone): 
        super().__init__(name, email) 
        self.phone = phone 
```

This example first gets the instance of the parent object using `super`, and calls `__init__` on that object, passing in the expected arguments. It then does its own initialization, namely, setting the `phone` attribute.

A `super()` call can be made inside any method. Therefore, all methods can be modified via overriding and calls to `super`. The call to `super` can also be made at any point  in the method; we don't have to make the call as the first line. For  example, we may need to manipulate or validate incoming parameters  before forwarding them to the superclass.