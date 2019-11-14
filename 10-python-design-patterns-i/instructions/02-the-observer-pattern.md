The observer pattern is useful  for state monitoring and event handling situations. This pattern allows  a given object to be monitored by an unknown and dynamic group of **observer** objects.

Whenever a value on the core object changes, it lets all the observer objects know that a change has occurred, by calling an `update()`  method. Each observer may be responsible for different tasks whenever  the core object changes; the core object doesn't know or care what those  tasks are, and the observers don't typically know or care what other  observers are doing.

Here it is in UML:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/62e8018a-3452-4108-b6c0-355ca5ec8b97.png)

# An Observer Example

The observer pattern might be useful in a redundant  backup system. We can write a core object that maintains certain  values, and then have one or more observers create serialized copies of  that object. These copies might be stored in a database, on a remote  host, or in a local file, for example. Let's implement the core object  using properties:

```python
class Inventory:
    def __init__(self):
        self.observers = []
        self._product = None
        self._quantity = 0

    def attach(self, observer):
        self.observers.append(observer)

    @property
    def product(self):
        return self._product

    @product.setter
    def product(self, value):
        self._product = value
        self._update_observers()

    @property
    def quantity(self):
        return self._quantity

    @quantity.setter
    def quantity(self, value):
        self._quantity = value
        self._update_observers()

    def _update_observers(self):
        for observer in self.observers:
            observer()
```

This object has two properties that, when set, call the `_update_observers`  method on itself. All this method does is loop over any registered  observers and let each know that something has changed. In this case, we  call the observer object directly; the object will have to implement `__call__`  to process the update. This would not be possible in many  object-oriented programming languages, but it's a useful shortcut in  Python that can help make our code more readable.

Now let's implement a simple observer object; this one will just print out some state to the console:

```python
class ConsoleObserver: 
    def __init__(self, inventory): 
        self.inventory = inventory 
 
    def __call__(self): 
        print(self.inventory.product) 
        print(self.inventory.quantity) 
```

There's  nothing terribly exciting here; the observed object is set up in the  initializer, and when the observer is called, we do **something**. We can test the observer in an interactive console:

```python
>>> i = Inventory()>>> c = ConsoleObserver(i)>>> i.attach(c)>>> i.product = "Widget"Widget0>>> i.quantity = 5Widget5
```

After attaching the observer to the `Inventory`  object, whenever we change one of the two observed properties, the  observer is called and its action is invoked. We can even add two  different observer instances:

```python
>>> i = Inventory()>>> c1 = ConsoleObserver(i)>>> c2 = ConsoleObserver(i)>>> i.attach(c1)>>> i.attach(c2)>>> i.product = "Gadget"Gadget0Gadget0
```

This  time when we change the product, there are two sets of output, one for  each observer. The key idea here is that we can easily add totally  different types of observers that back up the data in a file, database,  or internet application at the same time.

The observer pattern  detaches the code being observed from the code doing the observing. If  we were not using this pattern, we would have had to put code in each of  the properties  to handle the different cases that might come up; logging to the  console, updating a database or file, and so on. The code for each of  these tasks would all be mixed in with the observed object. Maintaining  it would be a nightmare, and adding new monitoring functionality at a  later date would be painful.