The abstract factory pattern is normally used when  we have multiple possible implementations of a system that depend on  some configuration or platform issue. The calling code requests an  object from the abstract factory, not knowing exactly what class of  object will be returned. The underlying implementation returned may  depend on a variety of factors, such as current locale, operating  system, or local configuration.

Common examples of the abstract  factory pattern include code for operating-system-independent toolkits,  database backends, and country-specific formatters or calculators. An  operating-system-independent GUI toolkit might use an abstract factory  pattern that returns a set of WinForm widgets under Windows, Cocoa  widgets under Mac, GTK widgets under Gnome, and QT widgets under KDE.  Django provides an abstract factory that returns a set of object  relational classes for interacting with a specific database backend  (MySQL, PostgreSQL, SQLite, and others) depending on a configuration  setting for the current site. If the application needs to be deployed in  multiple places, each one can use a different database backend by  changing only one configuration variable. Different countries have  different systems for calculating taxes, subtotals, and totals on retail  merchandise; an abstract factory can return a particular tax  calculation object.

The UML class diagram for an abstract factory  pattern is hard to understand without a specific example, so let's turn  things around and create a concrete example first. In our example, we'll  create a set of formatters that depend on a specific locale and help us  format dates and currencies. There will be an abstract factory class  that picks the specific factory, as well as a couple of example concrete  factories, one for France and one for the USA. Each of these will  create formatter objects for dates and times, which can be queried to  format a specific value. This is depicted in the following diagram:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/6a4ba21f-c95d-4e3c-aabf-7a2fd9e65b8d.png)

Comparing that image to the earlier, simpler text shows that a  picture is not always worth a thousand words, especially considering we  haven't even allowed for factory selection code here.

Of course, in Python, we don't have to implement any interface classes, so we can discard `DateFormatter`, `CurrencyFormatter`, and `FormatterFactory`. The formatting classes themselves are pretty straightforward, if verbose, shown here:

```python
class FranceDateFormatter:
    def format_date(self, y, m, d):
        y, m, d = (str(x) for x in (y, m, d))
        y = "20" + y if len(y) == 2 else y
        m = "0" + m if len(m) == 1 else m
        d = "0" + d if len(d) == 1 else d
        return "{0}/{1}/{2}".format(d, m, y)


class USADateFormatter:
    def format_date(self, y, m, d):
        y, m, d = (str(x) for x in (y, m, d))
        y = "20" + y if len(y) == 2 else y
        m = "0" + m if len(m) == 1 else m
        d = "0" + d if len(d) == 1 else d
        return "{0}-{1}-{2}".format(m, d, y)


class FranceCurrencyFormatter:
    def format_currency(self, base, cents):
        base, cents = (str(x) for x in (base, cents))
        if len(cents) == 0:
            cents = "00"
        elif len(cents) == 1:
            cents = "0" + cents

        digits = []
        for i, c in enumerate(reversed(base)):
            if i and not i % 3:
                digits.append(" ")
            digits.append(c)
        base = "".join(reversed(digits))
        return "{0}€{1}".format(base, cents)


class USACurrencyFormatter:
    def format_currency(self, base, cents):
        base, cents = (str(x) for x in (base, cents))
        if len(cents) == 0:
            cents = "00"
        elif len(cents) == 1:
            cents = "0" + cents
        digits = []
        for i, c in enumerate(reversed(base)):
            if i and not i % 3:
                digits.append(",")
            digits.append(c)
        base = "".join(reversed(digits))
        return "${0}.{1}".format(base, cents)
```

These classes use some basic string manipulation to try to turn a variety of possible inputs (integers, strings of different lengths, and others) into the following formats:

|              | **USA**    | **France** |
| ------------ | ---------- | ---------- |
| **Date**     | mm-dd-yyyy | dd/mm/yyyy |
| **Currency** | $14,500.50 | 14 500€50  |

There could obviously be more validation on the input in this code, but let's keep it simple for this example.

Now that we have the formatters set up, we just need to create the formatter factories, as follows:

```python
class USAFormatterFactory:
    def create_date_formatter(self):
        return USADateFormatter()

    def create_currency_formatter(self):
        return USACurrencyFormatter()


class FranceFormatterFactory:
    def create_date_formatter(self):
        return FranceDateFormatter()

    def create_currency_formatter(self):
        return FranceCurrencyFormatter()
```

Now we set  up the code that picks the appropriate formatter. Since this is the  kind of thing that only needs to be set up once, we could make it a  singleton–except singletons aren't very useful in Python. Let's just  make the current formatter a module-level variable instead:

```python
country_code = "US"
factory_map = {"US": USAFormatterFactory, "FR": FranceFormatterFactory}
formatter_factory = factory_map.get(country_code)()
```

In  this example, we hardcode the current country code; in practice, it  would likely introspect the locale, the operating system, or a  configuration file to choose the code. This example uses a dictionary to  associate the country codes with factory classes. Then, we grab the  correct class from the dictionary and instantiate it.

It is easy  to see what needs to be done when we want to add support for more  countries: create the new formatter classes and the abstract factory  itself. Bear in mind that `Formatter` classes  might be reused; for example, Canada formats its currency the same way  as the USA, but its date format is more sensible than its Southern  neighbor.

Abstract factories often return a singleton object, but  this is not required. In our code, it's returning a new instance of each  formatter every time it's called. There's no reason the formatters  couldn't be stored as instance variables and the same instance returned  for each factory.

Looking back at these examples, we see that,  once again, there appears to be a lot of boilerplate code for factories  that just doesn't feel necessary in Python. Often, the requirements that  might call for an abstract factory can be more easily fulfilled by  using a separate module for each factory type (for example: the USA and  France), and then ensuring that the correct module is being accessed in a  factory module. The package structure for such modules might look  like this:

```python
localize/ 
    __init__.py 
    backends/ 
        __init__.py 
        USA.py 
        France.py 
        ... 
```

The trick is that `__init__.py` in the `localize` package can contain logic that redirects all requests to the correct backend. There are a variety of ways this might be done.

If we know that the backend is never going to change dynamically (that is, without a program restart), we can just put some `if` statements in `__init__.py` that check the current country code, and use the (normally unacceptable) `from``.backends.USA``import``*` syntax to import all variables from the appropriate backend. Or, we could import each of the backends and set a `current_backend` variable to point at a specific module, as follows:

```python
from .backends import USA, France 
 
if country_code == "US": 
    current_backend = USA 
```

Depending on which solution we choose, our client code would have to call either `localize.format_date` or `localize.current_backend.format_date` to get a date formatted  in the current country's locale. The end result is much more Pythonic  than the original abstract factory pattern and, in typical usage, is  just as flexible.