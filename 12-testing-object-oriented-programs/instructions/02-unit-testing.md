Let's start our exploration with Python's built-in test library. This library provides a common object-oriented interface for **unit tests**. Unit tests focus on testing the least amount of code possible in any one test. Each one tests a single unit of the total amount of available code.

The Python library for this is called, unsurprisingly, `unittest`. It provides several tools for creating and running unit tests, the most important being the `TestCase`  class. This class provides a set of methods that allow us to compare  values, set up tests, and clean up when they have finished.

When we want to write a set of unit tests for a specific task, we create a subclass of `TestCase` and write individual methods to do the actual testing. These methods must all start with the name `test`.  When this convention is followed, the tests automatically run as part  of the test process. Normally, the tests set some values on an object  and then run a method, and use the built-in comparison methods to ensure  that the right results were calculated. Here's a very simple example:

```python
import unittest


class CheckNumbers(unittest.TestCase):
    def test_int_float(self):
        self.assertEqual(1, 1.0)


if __name__ == "__main__":
    unittest.main()
```

This code simply subclasses the `TestCase` class and adds a method that calls the `TestCase.assertEqual`  method. This method will either succeed or raise an exception,  depending on whether the two parameters are equal. If we run this code,  the `main` function from `unittest` will give us the following output:

```markdown
.--------------------------------------------------------------Ran 1 test in 0.000sOK
```

Did you know that floats and integers can be compared as equal? Let's add a failing test, as follows:

```python
    def test_str_float(self): 
        self.assertEqual(1, "1") 
```

The output of this code is more sinister, as integers and strings are not considered equal:

```markup
.F============================================================FAIL: test_str_float (__main__.CheckNumbers)--------------------------------------------------------------Traceback (most recent call last):  File "first_unittest.py", line 9, in test_str_float    self.assertEqual(1, "1")AssertionError: 1 != '1'--------------------------------------------------------------Ran 2 tests in 0.001sFAILED (failures=1)
```

The dot on the first line indicates that the first test (the one we wrote before) passed successfully; the letter `F`  after it shows that the second test failed. Then, at the end, it gives  us some informative output telling us how and where the test failed,  along with a summary of the number of failures.

We can have as many test methods on one `TestCase` class as we like. As long as the method name begins with `test`,  the test runner will execute each one as a separate, isolated test.  Each test should be completely independent of other tests. Results or  calculations from a previous test should have no impact on the current  test. The key to writing good unit tests is keeping each  test method as short as possible, testing a small unit of code with each  test case. If our code does not seem to naturally break up into such testable units, it's probably a sign that the code needs to be redesigned.

# Assertion Methods

The general layout of a test case is to set certain variables to known values, run one or more functions, methods, or processes, and then **prove** that correct expected results were returned or calculated by using `TestCase` assertion methods.

There are a few different assertion methods available to confirm that specific results have been achieved. We just saw `assertEqual`, which will cause a test failure if the two parameters do not pass an equality check. The inverse, `assertNotEqual`, will fail if the two parameters do compare as equal. The `assertTrue` and `assertFalse` methods each accept a single expression, and fail if the expression does not pass an `if` test. These tests do not check for the Boolean values `True` or `False`. Rather, they test the same condition as though an `if` statement were used: `False`, `None`, `0`, or an empty list, dictionary, string, set, or tuple would pass a call to the `assertFalse` method. Nonzero numbers, containers with values in them, or the value `True` would succeed when calling the `assertTrue` method.

There is an `assertRaises`  method that can be used to ensure that a specific function call raises a  specific exception or, optionally, it can be used as a context manager  to wrap inline code. The test passes if the code inside the `with` statement raises the proper exception; otherwise, it fails. The following code snippet is an example of both versions:

```python
import unittest


def average(seq):
    return sum(seq) / len(seq)


class TestAverage(unittest.TestCase):
    def test_zero(self):
        self.assertRaises(ZeroDivisionError, average, [])

    def test_with_zero(self):
        with self.assertRaises(ZeroDivisionError):
            average([])


if __name__ == "__main__":
    unittest.main()
```

The context manager allows us to write the code the way we would  normally write it (by calling functions or executing code directly),  rather than having to wrap the function call in another function call.

There are also several other assertion methods, summarized in the following table:

| **Methods**                                                  | **Description**                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `assertGreater``assertGreaterEqual``assertLess``assertLessEqual` | Accept two comparable objects and ensure the named inequality holds. |
| `assertIn``assertNotIn`                                      | Ensure an element is (or is not) an element in a container object. |
| `assertIsNone``assertIsNotNone`                              | Ensure an element is (or is not) the exact`None` value (but not another falsey value). |
| `assertSameElements`                                         | Ensure two container objects have the same elements, ignoring the order. |
| `assertSequenceEqualassertDictEqual``assertSetEqual``assertListEqual``assertTupleEqual` | Ensure two containers have the same elements in the same order. If there's a failure, show a code difference comparing the two lists to see where they differ. The last four methods also test the type of the list. |

 

Each of the assertion methods accepts an optional argument named `msg.` If supplied, it is included in the error message if the assertion fails. This can be useful for clarifying what was expected or explaining where a bug may have occurred to cause the assertion to fail. I rarely use this syntax, however, preferring to use descriptive names for the test method instead.

# Reducing Boilerplate and Cleaning Up

After writing a few small tests, we often find that we have to write the same setup code for several related tests. For example, the following `list` subclass has three methods for statistical calculations:

```python
from collections import defaultdict 
 
class StatsList(list): 
    def mean(self): 
        return sum(self) / len(self) 
 
    def median(self): 
        if len(self) % 2: 
            return self[int(len(self) / 2)] 
        else: 
            idx = int(len(self) / 2) 
            return (self[idx] + self[idx-1]) / 2 
 
    def mode(self): 
        freqs = defaultdict(int) 
        for item in self: 
            freqs[item] += 1 
        mode_freq = max(freqs.values()) 
        modes = [] 
        for item, value in freqs.items(): 
            if value == mode_freq: 
                modes.append(item) 
        return modes 
```

Clearly, we're going to want  to test situations with each of these three methods that have very  similar inputs. We'll want to see what happens with empty lists, with  lists containing non-numeric values, or with lists containing a normal  dataset, for example. We can use the `setUp` method on the `TestCase` class to perform  initialization for each test. This method accepts no arguments, and  allows us to do arbitrary setup before each test is run. For example, we  can test all three methods on identical lists of integers as follows:

```python
from stats import StatsList
import unittest


class TestValidInputs(unittest.TestCase):
    def setUp(self):
        self.stats = StatsList([1, 2, 2, 3, 3, 4])

    def test_mean(self):
        self.assertEqual(self.stats.mean(), 2.5)

    def test_median(self):
        self.assertEqual(self.stats.median(), 2.5)
        self.stats.append(4)
        self.assertEqual(self.stats.median(), 3)

    def test_mode(self):
        self.assertEqual(self.stats.mode(), [2, 3])
        self.stats.remove(2)
        self.assertEqual(self.stats.mode(), [3])


if __name__ == "__main__":
    unittest.main()
```

If we run this example, it indicates that all tests pass. Notice first that the `setUp` method is never explicitly called inside the three `test_*` methods. The test suite does this on our behalf. More importantly, notice how `test_median` alters the list, by adding an additional `4` to it, yet when the subsequent `test_mode` is called, the list has returned to the values specified in `setUp`. If it had not, there would be two fours in the list, and the `mode` method would have returned three values. This demonstrates that `setUp`  is called individually before each test, ensuring the test class starts  with a clean slate. Tests can be executed in any order, and the results  of one test must never depend on any other tests.

In addition to the `setUp` method, `TestCase` offers a no-argument `tearDown`  method, which can be used for cleaning up after each and every test on  the class has run. This method is useful if cleanup requires anything  other than letting an object be garbage collected.

For example, if we are testing code that does file I/O, our tests may create new files as a side effect of testing. The `tearDown`  method can remove these files and ensure the system is in the same  state it was before the tests ran. Test cases should never have side  effects. In general, we group test methods into separate `TestCase`  subclasses depending on what setup code they have in common. Several  tests that require the same or similar setup will be placed in one  class, while tests that require unrelated setup go in another class.

# Organizing and Running Tests

It doesn't take long for a collection of unit tests to grow very large and unwieldy. It can quickly become  complicated to load and run all the tests at once. This is a primary  goal of unit testing: trivially run all tests on our program and get a  quick **yes or no** answer to the question, **did my recent changes break anything?**.

As  with normal program code, we should divide our test classes into  modules and packages that keep them organized. If you name each test  module starting with the four characters **test**, there's an easy way to find and run them all. Python's `discover` module looks for any modules in the current folder or subfolders with names that start with `test`. If it finds any `TestCase`  objects in these modules, the tests are executed. It's a painless way  to ensure we don't miss running any tests. To use it, ensure your test  modules are named `test_<something>.py` and then run the `python3``-m``unittest``discover `command.

Most Python programmers choose to put their tests in a separate package (usually named `tests/`  alongside their source directory). This is not required, however.  Sometimes it makes sense to put the test modules for different packages  in a subpackage next to that package, for example.

# Ignoring Broken Tests

Sometimes,  a test is known to fail, but we don't want the test suite to report the  failure. This may be because a broken or unfinished feature has tests  written, but we aren't currently focusing on improving it. More often,  it happens because a feature is only available on a certain platform,  Python version, or for advanced versions of a specific library. Python  provides us with a few decorators to mark tests as expected to fail or  to be skipped under known conditions.

These decorators are as follows:

- `expectedFailure()`
- `skip(reason)`
- `skipIf(condition, reason)`
- `skipUnless(condition, reason)`

These  are applied using the Python decorator syntax. The first one accepts no  arguments, and simply tells the test runner not to record the test as a  failure when it fails. The `skip` method  goes one step further and doesn't even bother to run the test. It  expects a single string argument describing why the test was skipped.  The other two decorators accept two arguments, one a Boolean expression  that indicates whether or not the test should be run, and a similar  description. In use, these three decorators might be applied as they are  in the following code:

```python
import unittest
import sys


class SkipTests(unittest.TestCase):
    @unittest.expectedFailure
    def test_fails(self):
        self.assertEqual(False, True)

    @unittest.skip("Test is useless")
    def test_skip(self):
        self.assertEqual(False, True)

    @unittest.skipIf(sys.version_info.minor == 4, "broken on 3.4")
    def test_skipif(self):
        self.assertEqual(False, True)

    @unittest.skipUnless(
        sys.platform.startswith("linux"), "broken unless on linux"
    )
    def test_skipunless(self):
        self.assertEqual(False, True)


if __name__ == "__main__":
    unittest.main()
```

The first test fails, but it is  reported as an expected failure; the second test is never run. The  other two tests may or may not be run depending on the current Python  version and operating system. On my Linux system, running Python 3.7,  the output looks as follows:

```markup
xssF
======================================================================
FAIL: test_skipunless (__main__.SkipTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "test_skipping.py", line 22, in test_skipunless
    self.assertEqual(False, True)
AssertionError: False != True

----------------------------------------------------------------------
Ran 4 tests in 0.001s

FAILED (failures=1, skipped=2, expected failures=1)
```

The `x` on the first line indicates an expected failure; the two `s` characters represent skipped tests, and the `F` indicates a real failure, since the conditional to `skipUnless` was `True` on my system.