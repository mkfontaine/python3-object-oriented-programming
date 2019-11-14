The flyweight pattern is a memory optimization  pattern. Novice Python programmers tend to ignore memory optimization,  assuming the built-in garbage collector will take care of them. This is  usually perfectly acceptable, but when developing larger applications  with many related objects, paying attention to memory concerns can have a  huge payoff.

The flyweight pattern ensures that objects that  share a state can use the same memory for that shared state. It is  normally implemented only after a program has demonstrated memory  problems. It may make sense to design an optimal configuration from the  beginning in some situations, but bear in mind that premature  optimization is the most effective way to create a program that is too  complicated to maintain.

Let's have a look at the following UML diagram for the flyweight pattern:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/b2edd33d-98e5-4d88-b246-b019fc9548a0.png)

Each **Flyweight** has no specific state. Any time it needs to perform an operation on **SpecificState**, that state needs to be passed into the **Flyweight**  by the calling code. Traditionally, the factory that returns a  flyweight is a separate object; its purpose is to return a flyweight for  a given key identifying that flyweight. It works like the singleton  pattern we discussed in **Python Design Patterns I**;  if the flyweight exists, we return it; otherwise, we create a new one.  In many languages, the factory is implemented, not as a separate object,  but as a static method on the `Flyweight` class itself.

Think  of an inventory system for car sales. Each individual car has a  specific serial number and is a specific color. But most of the details  about that car are the same for all cars of a particular model. For  example, the Honda Fit DX model is a bare-bones car with few features.  The LX model has A/C, tilt, cruise, and power windows and  locks. The Sport model has fancy wheels, a USB charger, and a spoiler.  Without the flyweight pattern, each individual car object would have to  store a long list of which features it did and did not have. Considering  the number of cars Honda sells in a year, this would add up to a huge amount of wasted memory.

Using  the flyweight pattern, we can instead have shared objects for the list  of features associated with a model, and then simply reference that  model, along with a serial number and color, for individual vehicles. In  Python, the flyweight factory is often implemented using that funky `__new__` constructor, similar to what we did with the singleton pattern.

Unlike the singleton pattern,  which only needs to return one instance of the class, we need to be  able to return different instances depending on the keys. We could store  the items in a dictionary and look them up based on the key. This  solution is problematic, however, because the item will remain in memory  as long as it is in the dictionary. If we sold out of LX model Fits,  the Fit flyweight would no longer be necessary, yet it would still be in the dictionary. We could clean this up whenever we sell a car, but isn't that what a garbage collector is for?

We can solve this by taking advantage of Python's `weakref` module. This module provides a `WeakValueDictionary`  object, which basically allows us to store items in a dictionary  without the garbage collector caring about them. If a value is in a weak  referenced dictionary and there are no other references to that object  stored anywhere in the application (that is, we sold out of LX models),  the garbage collector will eventually clean up for us.

Let's build the factory for our car flyweightsfirst, as follows:

```python
import weakref


class CarModel:
    _models = weakref.WeakValueDictionary()

    def __new__(cls, model_name, *args, **kwargs):
        model = cls._models.get(model_name)
        if not model:
            model = super().__new__(cls)
            cls._models[model_name] = model

        return model
```

Basically, whenever we construct a new flyweight with a given name, we first look up that name in the weak referenced dictionary; if it exists, we return that model; if not, we create a new one. Either way, we know the `__init__` method on the flyweight will be called every time, regardless of whether it is a new or existing object. Our `__init__` method can therefore look like the following code snippet:

```python
    def __init__(
        self,
        model_name,
        air=False,
        tilt=False,
        cruise_control=False,
        power_locks=False,
        alloy_wheels=False,
        usb_charger=False,
    ):
        if not hasattr(self, "initted"):
            self.model_name = model_name
            self.air = air
            self.tilt = tilt
            self.cruise_control = cruise_control
            self.power_locks = power_locks
            self.alloy_wheels = alloy_wheels
            self.usb_charger = usb_charger
            self.initted = True
```

The `if` statement ensures that we only initialize the object the first time `__init__`  is called. This means we can call the factory later with just the model  name and get the same flyweight object back. However, because the  flyweight will be garbage-collected if no external references to it  exist, we must be careful not to accidentally create a new flyweight  with null values.

Let's add a method to our flyweight that hypothetically looks up a serial number on a specific model of vehicle, and determines whether  it has been involved in any accidents. This method needs access to the  car's serial number, which varies from car to car; it cannot be stored  with the flyweight. Therefore, this data must be passed into the method  by the calling code, as follows:

```python
    def check_serial(self, serial_number):
        print(
            "Sorry, we are unable to check "
            "the serial number {0} on the {1} "
            "at this time".format(serial_number, self.model_name)
        )
```

We can define a class that stores the additional information, as well as a reference to the flyweight, as follows:

```python
class Car: 
    def __init__(self, model, color, serial): 
        self.model = model 
        self.color = color 
        self.serial = serial 
 
    def check_serial(self): 
        return self.model.check_serial(self.serial) 
```

We can also keep track of the available models, as well as the individual cars on the lot, as follows:

```python
>>> dx = CarModel("FIT DX")>>> lx = CarModel("FIT LX", air=True, cruise_control=True,... power_locks=True, tilt=True)>>> car1 = Car(dx, "blue", "12345")>>> car2 = Car(dx, "black", "12346")>>> car3 = Car(lx, "red", "12347")
```

Now, let's demonstrate the weak referencing at work in the following code snippet:

```python
>>> id(lx)3071620300>>> del lx>>> del car3>>> import gc>>> gc.collect()0>>> lx = CarModel("FIT LX", air=True, cruise_control=True,... power_locks=True, tilt=True)>>> id(lx)3071576140>>> lx = CarModel("FIT LX")>>> id(lx)3071576140>>> lx.airTrue
```

The `id`  function tells us the unique identifier for an object. When we call it a  second time, after deleting all references to the LX model and forcing  garbage collection, we see that the ID has changed. The value in the `CarModel __new__` factory dictionary was deleted and a fresh one was created. If we then try to construct a second `CarModel`  instance, however, it returns the same object (the IDs are the same),  and, even though we did not supply any arguments in the second call, the  `air` variable is still set to `True`. This means the object was not initialized the second time, just as we designed.

Obviously,  using the flyweight pattern is more complicated than just storing  features on a single car class. When should we choose to use it? The  flyweight pattern is designed for conserving memory; if we have hundreds  of thousands of similar objects, combining similar properties into a  flyweight can have an enormous impact on memory consumption.

It is  common for programming solutions that optimize CPU, memory, or disk  space to result in more complicated code than their unoptimized  brethren. It is therefore important to weigh up the trade-offs when  deciding between code maintainability and optimization. When choosing  optimization, try to use patterns such as flyweight to ensure that the  complexity introduced by optimization is confined to a single  (well-documented) section of the code.

info> If you have a lot of Python objects in one program, one of the quickest ways to save memory is through the use of `__slots__`. The `__slots__`  magic method is beyond the scope of this book, but there is plenty of  information available if you check online. If you are still low on  memory, flyweight may be a reasonable solution.