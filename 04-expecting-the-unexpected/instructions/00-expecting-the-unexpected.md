Programs are very fragile. It would be ideal if code always returned a  valid result, but sometimes a valid result can't be calculated. For  example, it's not possible to divide by zero, or to access the eighth  item in a five-item list.

In the old days, the only way around  this was to rigorously check the inputs for every function to make sure  they made sense. Typically, functions had special return values to  indicate an error condition; for example, they could return a negative  number to indicate that a positive value couldn't be calculated.  Different numbers might mean different errors occurred. Any code that  called this function would have to explicitly check for an error  condition and act accordingly. A lot of developers didn't bother to do  this, and programs simply crashed. However, in the object-oriented  world, this is not the case.

In this chapter, we will study **exceptions**,  special error objects that only need to be handled when it makes sense  to handle them. In particular, we will cover the following:

- How to cause an exception to occur
- How to recover when an exception has occurred
- How to handle different exception types in different ways
- Cleaning up when an exception has occurred
- Creating new types of exception
- Using the exception syntax for flow control