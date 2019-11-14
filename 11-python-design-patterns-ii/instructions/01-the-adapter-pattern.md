Unlike most of the patterns we reviewed  in the previous chapter, the adapter pattern is designed to interact  with existing code. We would not design a brand new set of objects that  implement the adapter pattern. Adapters are used to allow two  preexisting objects to work together, even if their interfaces are not  compatible. Like the display adapters that allow you to plug your Micro  USB charging cable into a USB-C phone, an adapter object sits between  two different interfaces, translating between them on the fly. The  adapter object's sole purpose is to perform this translation. Adapting  may entail a variety of tasks, such as converting arguments to a  different format, rearranging the order of arguments, calling a  differently named method, or supplying default arguments.

In  structure, the adapter pattern is similar to a simplified decorator  pattern. Decorators typically provide the same interface that they  replace, whereas adapters map between two different interfaces. This is  depicted in UML form in the following diagram:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/13d782e2-6229-4d45-9588-e20a0b0d4d9d.png)

Here, **Interface1** is expecting to call a method called **make_action(some, arguments)**. We already have this perfect **Interface2** class that does everything we want (and to avoid duplication, we don't want to rewrite it!), but it provides a method called **different_action(other, arguments)** instead. The **Adapter** class implements the **make_action** interface and maps the arguments to the existing interface.

The  advantage here is that the code that maps from one interface to another  is all in one place. The alternative would be really ugly; we'd have to  perform the translation in multiple places whenever we need to access  this code.

For example, imagine we have the following preexisting class, which takes a string date in the format `YYYY-MM-DD` and calculates a person's age on that date:

```python
class AgeCalculator:
    def __init__(self, birthday):
        self.year, self.month, self.day = (
            int(x) for x in birthday.split("-")
        )

    def calculate_age(self, date):
        year, month, day = (int(x) for x in date.split("-"))
        age = year - self.year
        if (month, day) < (self.month, self.day):
            age -= 1
        return age
```

This is a pretty simple class  that does what it's supposed to do. But we have to wonder what the  programmer was thinking, using a specifically formatted string instead  of using Python's incredibly useful built-in `datetime` library. As conscientious programmers who reuse code whenever possible, most of the programs we write will interact with `datetime` objects, not strings.

We have several options to address this scenario. We could rewrite the class to accept `datetime` objects, which would probably be more accurate anyway. But if this class had been provided by a third party and we don't know how to or can't change its internal structure, we need an alternative. We could use the class as it is, and whenever we want to calculate the age on a `datetime.date` object, we could call `datetime.date.strftime('%Y-%m-%d')` to convert it to the proper format. But that conversion would be happening in a lot of places, and worse, if we mistyped the `%m` as `%M`, it would give us the current minute instead of the  month entered.  Imagine if you wrote that in a dozen different places only to have to  go back and change it when you realized your mistake. It's not  maintainable code, and it breaks the DRY principle.

Instead, we can write an adapter that allows a normal date to be plugged into a normal `AgeCalculator`class, as shown in the following code:

```python
import datetime 


class DateAgeAdapter:
    def _str_date(self, date):
        return date.strftime("%Y-%m-%d")

    def __init__(self, birthday):
        birthday = self._str_date(birthday)
        self.calculator = AgeCalculator(birthday)

    def get_age(self, date):
        date = self._str_date(date)
        return self.calculator.calculate_age(date)
```

This adapter converts `datetime.date` and `datetime.time` (they have the same interface to `strftime`) into a string that our original `AgeCalculator` can use. Now we can use the original code with our new interface. I changed the method signature to `get_age`  to demonstrate that the calling interface may also be looking for a  different method name, not just a different type of argument.

Creating  a class as an adapter is the usual way to implement this pattern, but,  as usual, there are other ways to do it in Python. Inheritance and  multiple inheritance can be used to add functionality to a class. For  example, we could add an adapter on the `date` class so that it works with the original `AgeCalculator`class, as follows:

```python
import datetime 
class AgeableDate(datetime.date): 
    def split(self, char): 
        return self.year, self.month, self.day 
```

It's code like this that makes one wonder whether Python should even be legal. We have added a `split`  method to our subclass that takes a single argument (which we ignore)  and returns a tuple of year, month, and day. This works flawlessly with  the original `AgeCalculator` class because the code calls `strip` on a specially formatted string, and `strip`, in that case, returns a tuple of year, month, and day. The `AgeCalculator` code only cares if`strip` exists and returns acceptable values; it doesn't care if we really passed in a string. The following code really works:

```python
>>> bd = AgeableDate(1975, 6, 14)>>> today = AgeableDate.today()>>> todayAgeableDate(2015, 8, 4)>>> a = AgeCalculator(bd)>>> a.calculate_age(today)40
```

It  works but it's a stupid idea. In this particular instance, such an  adapter would be hard to maintain. We'd soon forget why we needed to add  a `strip` method to a `date`  class. The method name is ambiguous. That can be the nature of  adapters, but creating an adapter explicitly instead of using  inheritance usually clarifies its purpose.

Instead of inheritance, we can sometimes also use monkey-patching to add a method to an existing class. It won't work with the `datetime` object, as it doesn't allow attributes to be added at runtime.  In normal classes, however, we can just add a new method that provides the adapted interface that is required by calling code. Alternatively, we could extend or monkey-patch the `AgeCalculator` itself to replace the `calculate_age` method with something more amenable to our needs.

Finally,  it is often possible to use a function as an adapter; this doesn't  obviously fit the actual design of the adapter pattern, but if we recall  that functions are essentially objects with a `__call__` method, it becomes an obvious adapter adaptation.