Multiple inheritance is a touchy subject. In principle, it's simple: a  subclass that inherits from more than one parent class is able to  access functionality from both of them. In practice, this is less useful than it sounds and many expert programmers recommend against using it.

info> As  a humorous rule of thumb, if you think you need multiple inheritance,  you're probably wrong, but if you know you need it, you might be right.

The simplest and most useful form of multiple inheritance is called a **mixin**. A mixin  is a superclass that is not intended to exist on its own, but is meant  to be inherited by some other class to provide extra functionality. For  example, let's say we wanted to add functionality to our `Contact` class that allows sending an email to `self.email`.  Sending email is a common task that we might want to use on many other  classes. So, we can write a simple mixin class to do the emailing for  us:

```python
class MailSender: 
    def send_mail(self, message): 
        print("Sending mail to " + self.email) 
        # Add e-mail logic here 
```

For brevity, we won't include the actual email logic here; if you're interested in studying how it's done, see the `smtplib` module in the Python standard library.

This  class doesn't do anything special (in fact, it can barely function as a  standalone class), but it does allow us to define a new class that describes both a `Contact` and a `MailSender`, using multiple inheritance:

```python
class EmailableContact(Contact, MailSender): 
    pass 
```

The syntax for multiple inheritance looks  like a parameter list in the class definition. Instead of including one  base class inside the parentheses, we include two (or more), separated  by a comma. We can test this new hybrid to see the mixin at work:

```python
>>> e = EmailableContact("John Smith", "jsmith@example.net")>>> Contact.all_contacts[<__main__.EmailableContact object at 0xb7205fac>]>>> e.send_mail("Hello, test e-mail here")Sending mail to jsmith@example.net
```

The `Contact` initializer is still adding the new contact to the `all_contacts` list, and the mixin is able to send mail to `self.email`, so we know that everything is working.

This  wasn't so hard, and you're probably wondering what the dire warnings  about multiple inheritance are. We'll get into the complexities in a  minute, but let's consider some other options we had for this example,  rather than using a mixin:

- We could have used single inheritance and added the `send_mail`  function to the subclass. The disadvantage here is that the email  functionality then has to be duplicated for any other classes that need  an email.
- We can create a  standalone Python function for sending an email, and just call that  function with the correct email address supplied as a parameter when the  email needs to be sent (this would be my choice).
- We could have explored a few ways of using composition instead of inheritance. For example, `EmailableContact` could have a `MailSender` object as a property instead of inheriting from it.
- We could monkey patch (we'll briefly cover monkey patching in **Python Object-Oriented Shortcuts**) the `Contact` class to have a `send_mail` method after the class has been created. This is done by defining a function that accepts the `self` argument, and setting it as an attribute on an existing class.

Multiple inheritance works all right when mixing  methods from different classes, but it gets very messy when we have to  call methods on the superclass. There are multiple superclasses. How do  we know which one to call? How do we know what order to call them in?

Let's explore these questions by adding a home address to our `Friend`  class. There are a few approaches we might take. An address is a  collection of strings representing the street, city, country, and other  related details of the contact. We could pass each of these strings as a  parameter into the `Friend` class's `__init__` method. We could also store these strings in a tuple, dictionary, or dataclass (we'll discuss dataclasses in **Python Data Structures**) and pass them into `__init__` as a single argument. This is probably the best course of action if there are no methods that need to be added to the address.

Another option would be to create a new `Address` class to hold those strings together, and then pass an instance of this class into the `__init__` method in our `Friend`  class. The advantage of this solution is that we can add behavior (say,  a method to give directions or to print a map) to the data instead of  just storing it statically. This is an example of composition, as we  discussed in **Object-Oriented Design**. The has a relationship of composition is a perfectly viable solution to this problem and allows us to reuse `Address` classes in other entities, such as buildings, businesses, or organizations.

However,  inheritance is also a viable solution, and that's what we want to  explore. Let's add a new class that holds an address. We'll call this  new class `AddressHolder` instead of `Address` because inheritance defines an is a relationship. It is not correct to say a `Friend` class is an `Address` class, but since a friend can have an `Address` class, we can argue that a `Friend` class is an `AddressHolder` class. Later, we could  create other entities (companies, buildings) that also hold addresses.  Then again, such convoluted naming is a decent indication we should be  sticking with composition, rather than inheritance. But for pedagogical  purposes, we'll stick with inheritance. Here's our `AddressHolder` class:

```python
class AddressHolder: 
    def __init__(self, street, city, state, code): 
        self.street = street 
        self.city = city 
        self.state = state 
        self.code = code 
```

We just take all the data and toss it into instance variables upon initialization.

# The Diamond Problem

We can use multiple inheritance to add this new class as a parent of our existing `Friend` class. The tricky part is that we now have two parent `__init__`  methods, both of which need to be initialized. And they need to be  initialized with different arguments. How do we do this? Well, we could  start with a naïve approach:

```python
class Friend(Contact, AddressHolder): 
    def __init__( 
        self, name, email, phone, street, city, state, code): 
        Contact.__init__(self, name, email) 
        AddressHolder.__init__(self, street, city, state, code) 
        self.phone = phone 
```

In this example, we directly call the `__init__` function on each of the superclasses and explicitly pass the `self` argument. This example technically works; we can access the different variables directly on the class. But there are a few problems.

First,  it is possible for a superclass to go uninitialized if we neglect to  explicitly call the initializer. That wouldn't break this example, but  it could cause hard-to-debug program crashes in common scenarios.  Imagine trying to insert data into a database that has not been  connected to, for example.

A more insidious possibility is a  superclass being called multiple times because of the organization of  the class hierarchy. Look at this inheritance diagram:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/047bbf50-fdfb-413d-b8ff-e99eea630c17.png)

The `__init__` method from the `Friend` class first calls `__init__` on `Contact`, which implicitly initializes the `object` superclass (remember, all classes derive from `object`). `Friend` then calls `__init__` on `AddressHolder`, which implicitly initializes the `object` superclass *again*. This means the parent class has been set up twice. With the `object`  class, that's relatively harmless, but in some situations, it could  spell disaster. Imagine trying to connect to a database twice for every  request!

The base class should only be called once. Once, yes, but when? Do we call `Friend`, then `Contact`, then `Object`, and then `AddressHolder`? Or `Friend`, then `Contact`, then `AddressHolder`, and then `Object`?

info> The order in which methods can be called can be adapted on the fly by modifying the `__mro__` (**Method Resolution Order**) attribute on the class. This is beyond the scope of this course.

Let's look at a second contrived example, which illustrates this problem more clearly. Here, we have a base class that has a method named `call_me`.  Two subclasses override that method, and then another subclass extends  both of these using multiple inheritance. This is called diamond  inheritance because of the diamond shape of the class diagram:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/e7a8f08d-b401-4aaf-94f0-23f6ec785c5e.png)

Let's convert this diagram to code; this example shows when the methods are called:

```python
class BaseClass:
    num_base_calls = 0

    def call_me(self):
        print("Calling method on Base Class")
        self.num_base_calls += 1


class LeftSubclass(BaseClass):
    num_left_calls = 0

    def call_me(self):
        BaseClass.call_me(self)
        print("Calling method on Left Subclass")
        self.num_left_calls += 1


class RightSubclass(BaseClass):
    num_right_calls = 0

    def call_me(self):
        BaseClass.call_me(self)
        print("Calling method on Right Subclass")
        self.num_right_calls += 1


class Subclass(LeftSubclass, RightSubclass):
    num_sub_calls = 0

    def call_me(self):
        LeftSubclass.call_me(self)
        RightSubclass.call_me(self)
        print("Calling method on Subclass")
        self.num_sub_calls += 1
```

This example ensures that each overridden `call_me`  method directly calls the parent method with the same name. It lets us  know each time a method is called by printing the information to the  screen. It also updates a static variable on the class to show how many times it has been called. If we instantiate one `Subclass` object and call the method on it once, we get the output:

```python
>>> s = Subclass()>>> s.call_me()Calling method on Base ClassCalling method on Left SubclassCalling method on Base ClassCalling method on Right SubclassCalling method on Subclass>>> print(... s.num_sub_calls,... s.num_left_calls,... s.num_right_calls,... s.num_base_calls)1 1 1 2
```

Thus, we can clearly see the base class's `call_me`  method being called twice. This could lead to some pernicious bugs if  that method is doing actual work, such as depositing into a bank  account, twice.

The thing to keep in mind with multiple inheritance is that we only want to call the `next` method in the class hierarchy, not the `parent` method. In fact, that next method may not be on a parent or ancestor of the current class. The `super` keyword comes to our rescue once again. Indeed, `super` was originally developed to make complicated forms of multiple inheritance possible. Here is the same code written using `super`:

```python
class BaseClass:
    num_base_calls = 0

    def call_me(self):
        print("Calling method on Base Class")
        self.num_base_calls += 1


class LeftSubclass(BaseClass):
    num_left_calls = 0

    def call_me(self):
        super().call_me()
        print("Calling method on Left Subclass")
        self.num_left_calls += 1


class RightSubclass(BaseClass):
    num_right_calls = 0

    def call_me(self):
        super().call_me()
        print("Calling method on Right Subclass")
        self.num_right_calls += 1


class Subclass(LeftSubclass, RightSubclass):
    num_sub_calls = 0

    def call_me(self):
 super().call_me()
        print("Calling method on Subclass")
        self.num_sub_calls += 1
```

The change is pretty minor; we only replaced the naive direct calls with calls to `super()`, although the bottom subclass only calls `super` once rather than having to make the calls for both the left and right. The change is easy enough, but look at the difference when we execute it:

```python
>>> s = Subclass()>>> s.call_me()Calling method on Base ClassCalling method on Right SubclassCalling method on Left SubclassCalling method on Subclass>>> print(s.num_sub_calls, s.num_left_calls, s.num_right_calls,s.num_base_calls)1 1 1 1
```

Looks good; our base method is only being called once. But what is `super()` actually doing here? Since the `print` statements are executed after the `super`  calls, the printed output is in the order each method is actually  executed. Let's look at the output from back to front to see who is  calling what.

First, `call_me` of `Subclass` calls `super().call_me()`, which happens to refer to `LeftSubclass.call_me()`. The `LeftSubclass.call_me()` method then calls `super().call_me()`, but in this case, `super()` is referring to `RightSubclass.call_me()`.

**Pay particular attention to this**: the `super` call is *not* calling the method on the superclass of `LeftSubclass` (which is `BaseClass`). Rather, it is calling `RightSubclass`, even though it is not a direct parent of `LeftSubclass`! This is the **next** method, not the parent method. `RightSubclass` then calls `BaseClass` and the `super` calls have ensured each method in the class hierarchy is executed once.

# Different Sets of Arguments

This is going to make things complicated as we return to our `Friend` multiple inheritance example. In the `__init__` method for `Friend`, we were originally calling `__init__` for both parent classes, **with different sets of arguments**:

```python
Contact.__init__(self, name, email) 
AddressHolder.__init__(self, street, city, state, code)
```

How can we manage different sets of arguments when using `super`? We don't necessarily know which class `super` is going to try to initialize first. Even if we did, we need a way to pass the `extra` arguments so that subsequent calls to `super`, on other subclasses, receive the right arguments.

Specifically, if the first call to `super` passes the `name` and `email` arguments to `Contact.__init__`, and `Contact.__init__` then calls `super`, it needs to be able to pass the address-related arguments to the `next` method, which is `AddressHolder.__init__`.

This  problem manifests itself anytime we want to call superclass methods  with the same name, but with different sets of arguments. Most often,  the only time you would want to call a superclass with a completely  different set of arguments is in `__init__`,  as we're doing here. Even with regular methods, though, we may want to  add optional parameters that only make sense to one subclass or set of  subclasses.

Sadly, the only way to solve this problem is to plan  for it from the beginning. We have to design our base class parameter  lists to accept keyword arguments for any parameters that are not  required by every subclass implementation. Finally, we must ensure the  method freely accepts unexpected arguments and passes them on to its `super` call, in case they are necessary to later methods in the inheritance order.

Python's  function parameter syntax provides all the tools we need to do this,  but it makes the overall code look cumbersome. Have a look at the proper version of the `Friend` multiple inheritance code, as follows:

```python
class Contact:
    all_contacts = []

    def __init__(self, name="", email="", **kwargs):
        super().__init__(**kwargs)
        self.name = name
        self.email = email
        self.all_contacts.append(self)


class AddressHolder:
    def __init__(self, street="", city="", state="", code="", **kwargs):
        super().__init__(**kwargs)
        self.street = street
        self.city = city
        self.state = state
        self.code = code


class Friend(Contact, AddressHolder):
    def __init__(self, phone="", **kwargs):
        super().__init__(**kwargs)
        self.phone = phone
```

We've changed all arguments to keyword arguments by giving them an empty string as a default value. We've also ensured that a `**kwargs`  parameter is included to capture any additional parameters that our  particular method doesn't know what to do with. It passes these parameters up to the next class with the `super` call.

info> If you aren't familiar with the `**kwargs`  syntax, it basically collects any keyword arguments passed into the  method that were not explicitly listed in the parameter list. These  arguments are stored in a dictionary named `kwargs` (we can call the variable whatever we like, but convention suggests `kw`, or `kwargs`). When we call a different method (for example, `super().__init__`) with a `**kwargs`  syntax, it unpacks the dictionary and passes the results to the method  as normal keyword arguments. We'll cover this in detail in **Python Object-Oriented Shortcuts**.

The  previous example does what it is supposed to do. But it's starting to  look messy, and it is difficult to answer the question, **What arguments do we need to pass into** `Friend.__init__`?  This is the foremost question for anyone planning to use the class, so a  docstring should be added to the method to explain what is happening.

Furthermore, even this implementation is insufficient if we want to **reuse** variables in parent classes. When we pass the `**kwargs` variable to `super`, the dictionary does not include any of the variables that were included as explicit keyword arguments. For example, in `Friend.__init__`, the call to `super` does not have `phone` in the `kwargs` dictionary. If any of the other classes need the `phone`  parameter, we need to ensure it is in the dictionary that is passed.  Worse, if we forget to do this, it will be extremely frustrating to  debug because the superclass will not complain, but will simply assign  the default value (in this case, an empty string) to the variable.

There are a few ways to ensure that the variable is passed upward. Assume the `Contact` class does, for some reason, need to be initialized with a `phone` parameter, and the `Friend` class will also need access to it. We can do any of the following:

- Don't include `phone` as an explicit keyword argument. Instead, leave it in the `kwargs` dictionary. `Friend` can look it up using the `kwargs['phone'] ` syntax. When it passes `**kwargs` to the `super` call, `phone` will still be in the dictionary.
- Make `phone` an explicit keyword argument, but update the `kwargs` dictionary before passing it to `super`, using the standard dictionary `kwargs['phone'] = phone` syntax.
- Make `phone` an explicit keyword argument, but update the `kwargs` dictionary using the `kwargs.update` method. This is useful if you have several arguments to update. You can create the dictionary passed into `update` using either the `dict(phone=phone)` constructor, or the dictionary `{'phone': phone}` syntax.
- Make `phone` an explicit keyword argument, but pass it to the super call explicitly with the `super().__init__(phone=phone, **kwargs)` syntax.

We have covered many of the caveats involved  with multiple inheritance in Python. When we need to account for all  possible situations, we have to plan for them and our code will get  messy. Basic multiple inheritance can be handy but, in many cases, we  may want to choose a more transparent way of combining two disparate  classes, usually using composition or one of the design patterns we'll  be covering in **Design Patterns I** and **Design Patterns II**.

info> I  have wasted entire days of my life trawling through complex multiple  inheritance hierarchies trying to figure out what arguments I need to  pass into one of the deeply nested subclasses. The author of the code  tended not to document his classes and often passed the kwargs—Just in  case they might be needed someday. This was a particularly bad example  of using multiple inheritance when it was not needed. Multiple  inheritance is a big fancy term that new coders like to show off, but I  recommend avoiding it, even when you think it's a good choice. Your  future self and other coders will be glad they understand your code when  they have to read it later.