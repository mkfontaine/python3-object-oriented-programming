Dataclasses are basically regular objects with a clean syntax for predefining attributes. There are a few ways to create one, and we'll explore each in this section.

The simplest way is to use a similar construct to that used for named tuples, as follows:

```python
from dataclasses import make_dataclass
Stock = make_dataclass("Stock", "symbol", "current", "high", "low")
stock = Stock("FB", 177.46, high=178.67, low=175.79)
```

Once  instantiated, the stock object can be used like any regular class. You  can access and update attributes and can even assign other arbitrary  attributes to the object, as follows:

```python
>>> stock
Stock(symbol='FB', current=177.46, high=178.67, low=175.79)
>>> stock.current
177.46
>>> stock.current=178.25
>>> stock
Stock(symbol='FB', current=178.25, high=178.67, low=175.79)
>>> stock.unexpected_attribute = 'allowed'
>>> stock.unexpected_attribute
'allowed'
```

At first glance, it seems like dataclasses don't give you much benefit over a normal object with an appropriate constructor:

```python
class StockRegClass:
    def __init__(self, name, current, high, low):
        self.name = name
        self.current = current
        self.high = high
        self.low = low

stock_reg_class = Stock("FB", 177.46, high=178.67, low=175.79)
```

The obvious benefit is that with `make_dataclass`,  you get to define the class in one line instead of six. If you look a  little closer, you'll see that the dataclass also gives you a much more  useful string representation than the regular version. It also provides an equality comparison for free. The following example compares the regular class to these dataclass features:

```python
>>> stock_reg_class
<__main__.Stock object at 0x7f506bf4ec50>
>>> stock_reg_class2 = StockRegClass("FB", 177.46, 178.67, 175.79)
>>> stock_reg_class2 == stock_reg_class
False
>>> stock2 = Stock("FB", 177.46, 178.67, 175.79)
>>> stock2 == stock
True
```

As we'll soon see, dataclasses  also have many other useful features. But first, let's look at an  alternative (and more common) way to define a dataclass. Refer to the following block of code:

```python
from dataclasses import dataclass

@dataclass
class StockDecorated:
    name: str
    current: float
    high: float
    low: float
```

If you haven't seen type hints  before, this syntax probably looks truly bizarre. These so-called  variable annotations were introduced to the language in Python 3.6. I'm  classifying type hints as **beyond the scope of this course**, so I'll leave you to do a web search if you want to find out more about them.  For now, just know that the preceding is truly legal Python syntax, and  that it works. You don't have to take my word for it; just run the code  and observe the lack of syntax errors!

info> If  you don't feel like using type hints or your attribute takes a value  with a complicated type or set of types, specify the type as `Any`. You can pull the `Any` type into your namespace using `from typing import Any`.

The `dataclass` function is applied  as a class decorator. We encountered decorators in a previous chapter  when we were discussing properties. I promised then that we'll go into  more detail about them in a future chapter. I'll keep that promise in  chapter 10. For now, just know that the syntax is required to generate a  dataclass.

Granted, this syntax isn't much less verbose than the regular class with `__init__`,  but it gives us access to several additional dataclass features. For  example, you can specify a default value for a dataclass. Perhaps the  market is currently closed and you don't know what the values for the  day are:

```python
@dataclass
class StockDefaults:
    name: str
    current: float = 0.0
    high: float = 0.0
    low: float = 0.0
```

You can construct this class  with just the stock name; the rest of the values will take on the  defaults. But you can still specify values if you prefer, as follows:

```python
>>> StockDefaults('FB')
StockDefaults(name='FB', current=0.0, high=0.0, low=0.0)
>>> StockDefaults('FB', 177.46, 178.67, 175.79)
StockDefaults(name='FB', current=177.46, high=178.67, low=175.79)
```

We saw earlier that dataclasses  automatically support equality comparison. If all the attributes compare  as equal, then the dataclass also compares as equal. By default,  dataclasses do not support other comparisons, such as less than or  greater than, and they can't be sorted. However, you can easily add  comparisons if you wish, demonstrated as follows:

```python
@dataclass(order=True)
class StockOrdered:
    name: str
    current: float = 0.0
    high: float = 0.0
    low: float = 0.0


stock_ordered1 = StockDecorated("FB", 177.46, high=178.67, low=175.79)
stock_ordered2 = StockOrdered("FB")
stock_ordered3 = StockDecorated("FB", 178.42, high=179.28, low=176.39)
```

All that we changed in this example was adding the `order=True` keyword to the dataclass constructor. But that gives us the opportunity to sort and compare the following values:

```python
>>> stock_ordered1 < stock_ordered2
False
>>> stock_ordered1 > stock_ordered2
True
>>> from pprint import pprint
>>> pprint(sorted([stock_ordered1, stock_ordered2, stock_ordered3]))
[StockOrdered(name='FB', current=0.0, high=0.0, low=0.0),
 StockOrdered(name='FB', current=177.46, high=178.67, low=175.79),
 StockOrdered(name='FB', current=178.42, high=179.28, low=176.39)]
```

When a dataclass receives the `order=True`  argument, it will, by default, compare the values based on each of the  attributes in the order they were defined. So, in this case, it first  compares the name on the two classes. If those are the same, it compares  the current price. If those are also the same, it will compare the  highs and then the lows. You can customize the sort order by providing a  `sort_index` attribute inside a `__post_init__`  method on the class, but I'll leave you to search the web to get the  full details of this and other advanced usages (such as immutability),  as this section is getting rather long and we have a lot of other data structures to study.