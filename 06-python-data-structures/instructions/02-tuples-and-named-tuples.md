Tuples are objects that can store a specific number of other objects in order. They are **immutable**,  meaning we can't add, remove, or replace objects on the fly. This may  seem like a massive restriction, but the truth is, if you need to modify  a tuple, you're using the wrong data type (usually, a list would be  more suitable). The primary benefit of tuples' immutability is that we  can use them as keys in dictionaries, and in other locations where an  object requires a hash value.

Tuples are used to store data;  behavior cannot be associated with a tuple. If we require behavior to  manipulate a tuple, we have to pass the tuple into a function (or method  on another object) that performs the action.

Tuples should  generally store values that are somehow different from each other. For  example, we would not put three stock symbols in a tuple, but we might  create a tuple containing a stock symbol with its current, high, and low prices for the day.  The primary purpose of a tuple is to aggregate different pieces of data  together into one container. Thus, a tuple can be the easiest tool to  replace the **object with no data** idiom.

We  can create a tuple by separating values with a comma. Usually, tuples  are wrapped in parentheses to make them easy to read and to separate  them from other parts of an expression, but this is not always  mandatory. The following two assignments are identical (they record a  stock, the current price, the high, and the low, for a rather profitable  company):

```python
>>> stock = "FB", 177.46, 178.67, 175.79>>> stock2 = ("FB", 177.46, 178.67, 175.79)
```

If  we're grouping a tuple inside of some other object, such as a function  call, list comprehension, or generator, the parentheses are required.  Otherwise, it would be impossible for the interpreter to know whether it  is a tuple or the next function parameter. For example, the following  function accepts a tuple and a date, and returns a tuple of the date and  the middle value between the stock's high and low value:

```python
import datetime


def middle(stock, date):
    symbol, current, high, low = stock
    return (((high + low) / 2), date)


mid_value, date = middle(
("FB", 177.46, 178.67, 175.79), datetime.date(2018, 8, 27)
)
```

The tuple is created directly inside the function  call by separating the values with commas and enclosing the entire  tuple in parentheses. This tuple is then followed by a comma to separate  it from the second argument.

This example also illustrates **tuple unpacking**. The first line inside the function unpacks the `stock`  parameter into four different variables. The tuple has to be exactly  the same length as the number of variables, or it will raise an  exception. We can also see an example of tuple unpacking in the last  clause, where the tuple returned from inside the function is unpacked  into two values, `mid_value` and `date`.  Granted, this is a strange thing to do, since we supplied the date to  the function in the first place, but it gave us a chance to see  unpacking at work.

Unpacking is a very useful feature in Python.  We can group variables together to make storing and passing them around  simpler, but the moment we need to access all of them, we can unpack  them into separate variables. Of course, sometimes we only need access  to one of the variables in the tuple. We can use the same syntax that we  use for other sequence types (lists and strings, for example) to access  an individual value:

```python
>>> stock = "FB", 75.00, 75.03, 74.90>>> high = stock[2]>>> high75.03
```

We can even use slice notation to extract larger pieces of tuples, as demonstrated in the following:

```python
>>> stock[1:3](75.00, 75.03)
```

These  examples, while illustrating how flexible tuples can be, also  demonstrate one of their major disadvantages: readability. How does  someone reading this code know what is in the second position of a  specific tuple? They can guess, from the name of the variable we  assigned it to, that it is `high` of some  sort, but if we had just accessed the tuple value in a calculation  without assigning it, there would be no such indication. They would have  to paw through the code to find where the tuple was declared before  they could discover what it does.

Accessing tuple members directly is fine in some circumstances, but don't make a habit of it. Such so-called **magic numbers**  (numbers that seem to come out of thin air with no apparent meaning  within the code) are the source of many coding errors and lead to hours  of frustrated debugging. Try to use tuples only when you know that all  the values are going to be useful at once and it's normally going to be  unpacked when it is accessed. If you have to access a member directly,  or by using a slice, and the purpose of that value is not immediately  obvious, at least include a comment explaining where it came from.

# Named Tuples

So, what do we do when we want to group values  together, but know we're frequently going to need to access them  individually? There are actually several options. We could use an empty  object, as discussed previously (but that is rarely useful, unless we  anticipate adding behavior later), or we could use a dictionary (most  useful if we don't know exactly how much data or which specific data  will be stored), as we'll cover in a later section. Two other options  are named tuples, which we'll discuss here, and dataclasses, in the next  section.

If we do not need to add behavior to the object, and we  know in advance which attributes we need to store, we can use a named  tuple. Named tuples are tuples with attitude. They are a great way to  group read-only data together.

Constructing a named tuple takes a bit more work than a normal tuple. First, we have to import `namedtuple`,  as it is not in the namespace by default. Then, we describe the named  tuple by giving it a name and outlining its attributes. This returns a  class-like object that we can instantiate with the required values as  many times as we want, as demonstrated in the following:

```python
from collections import namedtuple 
Stock = namedtuple("Stock", ["symbol", "current", "high", "low"])
stock = Stock("FB", 177.46, high=178.67, low=175.79) 
```

The `namedtuple`  constructor accepts two arguments. The first is an identifier for the  named tuple. The second is a list of string attributes that the named  tuple requires. The result is an object that can be called just like a  normal class to instantiate other objects. The constructor must have  exactly the correct number of arguments that can be passed in as  arguments or keyword arguments. As with normal objects, we can create as  many instances of this **class** as we like, with different values for each.

info> Be careful not to use a reserved keyword (class, for example) as an attribute for a named tuple.

The resulting `namedtuple` can then be packed, unpacked, indexed, sliced, and otherwise treated like a normal tuple, but we can also access individual attributes on it as if it were an object:

```python
>>> stock.high
175.79>>> symbol, current, high, low = stock>>> current177.46
```

info> Remember that creating named tuples is a two-step process. First, use `collections.namedtuple` to create a class, and then construct instances of that class.

Named tuples are perfect for many **data only** representations,  but they are not ideal for all situations. Like tuples and strings,  named tuples are immutable, so we cannot modify an attribute once it has  been set. For example, the current value of my company's stock has gone  down since we started this discussion, but we can't set the new value,  as can be seen in the following:

```python
>>> stock.current = 74.98Traceback (most recent call last):  File "<stdin>", line 1, in <module>AttributeError: can't set attribute
```

If we need to be able to change stored data, a dataclass may be what we need instead.