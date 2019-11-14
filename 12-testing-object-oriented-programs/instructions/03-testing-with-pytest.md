The Python `unittest` module requires a lot of boilerplate  code to set up and initialize tests. It is based on the very popular  JUnit testing framework for Java. It even uses the same method names  (you may have noticed they don't conform to the PEP-8 naming standard,  which suggests snake_case rather than CamelCase to indicate a method  name) and test layout. While this is effective for testing in Java, it's  not necessarily the best design for Python testing. I actually find the  `unittest` framework to be an excellent example of overusing object-oriented principles.

Because  Python programmers like their code to be elegant and simple, other test  frameworks have been developed, outside the standard library. Two of  the more popular ones are `pytest` and `nose`. The former is more robust and has had Python 3 support for much longer, so we'll discuss it here.

Since `pytest` is not part of the standard library, you'll need to download and install it yourself. You can get it from the `pytest` home page [here](<http://pytest.org/>).  The website has comprehensive installation instructions for a variety  of interpreters and platforms, but you can usually get away with the  more common Python package installer, pip. Just type `pip install pytest` on your command line and you'll be good to go.

`pytest` has a substantially different layout from the `unittest`  module. It doesn't require test cases to be classes. Instead, it takes  advantage of the fact that Python functions are objects, and allows any  properly named function to behave like a test. Rather than providing a  bunch of custom methods for asserting equality, it uses the `assert` statement to verify results. This makes tests more readable and maintainable.

When we run `pytest`, it starts in the current folder and searches for any modules or subpackages with names beginning with the characters `test_`. If any functions in this module also start with `test`, they will be executed as individual tests. Furthermore, if there are any classes in the module whose name starts with `Test`, any methods on that class that start with `test_` will also be executed in the test environment.

Using the following code, let's port the simplest possible `unittest` example we wrote earlier to `pytest`:

```python
def test_int_float(): 
    assert 1 == 1.0 
```

For the exact same test, we've written two lines of more readable code, in comparison to the six lines required in our first `unittest` example.

However,  we are not forbidden from writing class-based tests. Classes can be  useful for grouping related tests together or for tests that need to  access related attributes or methods on the class. The following example  shows an extended class with a passing and a failing test; we'll see  that the error output is more comprehensive than that provided by the `unittest` module:

```python
class TestNumbers: 
    def test_int_float(self): 
        assert 1 == 1.0 
 
    def test_int_str(self): 
        assert 1 == "1" 
```

Notice that the class doesn't have to extend any special objects to be picked up as a test (although `pytest` will run standard `unittest TestCases` just fine). If we run `pytest <filename>`, the output looks as follows:

```markup
============================== test session starts ==============================
platform linux -- Python 3.7.0, pytest-3.8.0, py-1.6.0, pluggy-0.7.1
rootdir: /home/dusty/Py3OOP/Chapter 12: Testing Object-oriented Programs, inifile:
collected 3 items

test_with_pytest.py ..F [100%]

=================================== FAILURES ====================================
___________________________ TestNumbers.test_int_str ____________________________

self = <test_with_pytest.TestNumbers object at 0x7fdb95e31390>

    def test_int_str(self):
> assert 1 == "1"
E AssertionError: assert 1 == '1'

test_with_pytest.py:10: AssertionError
====================== 1 failed, 2 passed in 0.03 seconds =======================
```

The  output starts with some useful information about the platform and  interpreter. This can be useful for sharing or discussing bugs across  disparate systems. The third line tells us the name of the file being  tested (if there are multiple test modules picked up, they will all be  displayed), followed by the familiar `.F` we saw in the `unittest` module; the `.` character indicates a passing test, while the letter `F` demonstrates a failure.

After all tests have run, the error output for each of them is displayed. It presents a summary of local variables (there is only one in this example: the `self`  parameter passed into the function), the source code where the error  occurred, and a summary of the error message. In addition, if an  exception other than an `AssertionError` is raised, `pytest` will present us with a complete traceback, including source code references.

By default, `pytest` suppresses output from `print` statements if the test is successful. This is useful for test debugging; when a test is failing, we can add `print`  statements to the test to check the values of specific variables and  attributes as the test runs. If the test fails, these values are output  to help with diagnosis. However, once the test is successful, the `print` statement output is not displayed, and they are easily ignored. We don't have to **clean up** output by removing `print` statements. If the tests ever fail again, due to future changes, the debugging output will be immediately available.

# One Way to do Setup and Cleanup

`pytest` supports setup and teardown methods similar to those used in `unittest`, but it provides even more flexibility. We'll discuss these briefly, since they are familiar, but they are not used as extensively as in the `unittest` module, as `pytest` provides us with a powerful fixtures facility, which we'll discuss in the next lesson.

If we are writing class-based tests, we can use two methods called `setup_method` and `teardown_method` in the same way that `setUp` and `tearDown` are called in `unittest`.  They are called before and after each test method in the class to  perform setup and cleanup duties. There is one difference from the `unittest` methods though. Both methods accept an argument: the function object representing the method being called.

In addition, `pytest` provides other setup and teardown functions to give us more control over when setup and cleanup code is executed. The `setup_class` and `teardown_class` methods are expected to be class methods; they accept a single argument (there is no `self`  argument) representing the class in question. These methods are only  run when the class is initiated rather than on each test run.

Finally, we have the `setup_module` and `teardown_module`  functions, which are run immediately before and after all tests (in  functions or classes) in that module. These can be useful for **one time** setup,  such as creating a socket or database connection that will be used by  all tests in the module. Be careful with this one, as it can  accidentally introduce dependencies between tests if the object stores  state that isn't correctly cleaned up between tests.

That short  description doesn't do a great job of explaining exactly when these  methods are called, so let's look at an example that illustrates exactly  when it happens:

```python
def setup_module(module):
    print("setting up MODULE {0}".format(module.__name__))


def teardown_module(module):
    print("tearing down MODULE {0}".format(module.__name__))


def test_a_function():
    print("RUNNING TEST FUNCTION")


class BaseTest:
    def setup_class(cls):
        print("setting up CLASS {0}".format(cls.__name__))

    def teardown_class(cls):
        print("tearing down CLASS {0}\n".format(cls.__name__))

    def setup_method(self, method):
        print("setting up METHOD {0}".format(method.__name__))

    def teardown_method(self, method):
        print("tearing down METHOD {0}".format(method.__name__))


class TestClass1(BaseTest):
    def test_method_1(self):
        print("RUNNING METHOD 1-1")

    def test_method_2(self):
        print("RUNNING METHOD 1-2")


class TestClass2(BaseTest):
    def test_method_1(self):
        print("RUNNING METHOD 2-1")

    def test_method_2(self):
        print("RUNNING METHOD 2-2")
```

The sole purpose of the `BaseTest`  class is to extract four methods that are otherwise identical to the  test classes, and use inheritance to reduce the amount of duplicate  code. So, from the point of view of `pytest`, the two subclasses have not only two test methods each, but also two setup and two teardown methods (one at the class level, one at the method level).

If we run these tests using `pytest` with the `print` function output suppression disabled (by passing the `-s` or `--capture=no` flag), they show us when the various functions are called in relation to the tests themselves:

```markup
setup_teardown.pysetting up MODULE setup_teardownRUNNING TEST FUNCTION.setting up CLASS TestClass1setting up METHOD test_method_1RUNNING METHOD 1-1.tearing down  METHOD test_method_1setting up METHOD test_method_2RUNNING METHOD 1-2.tearing down  METHOD test_method_2tearing down CLASS TestClass1setting up CLASS TestClass2setting up METHOD test_method_1RUNNING METHOD 2-1.tearing down  METHOD test_method_1setting up METHOD test_method_2RUNNING METHOD 2-2.tearing down  METHOD test_method_2tearing down CLASS TestClass2tearing down MODULE setup_teardown
```

The  setup and teardown methods for the module are executed at the beginning  and end of the session. Then the lone module-level test function is  run. Next, the setup method for the first class is executed, followed by  the two tests for that class. These tests are each individually wrapped  in separate `setup_method` and `teardown_method` calls. After the tests have executed, the teardown method on the class is called. The same sequence happens for the second class, before the `teardown_module` method is finally called, exactly once.

# A Completely Different Way to Set Up Variables

One of the most common  uses for the various setup and teardown functions is to ensure certain  class or module variables are available with a known value before each  test method is run.

`pytest` offers a completely different way of doing this, using what are known as **fixtures**.  Fixtures are basically named variables that are predefined in a test  configuration file. This allows us to separate configuration from the  execution of tests, and allows fixtures to be used across multiple classes and modules.

To  use them, we add parameters to our test function. The names of the  parameters are used to look up specific arguments in specially named  functions. For example, if we wanted to test the `StatsList` class we used while demonstrating `unittest`,  we would again want to repeatedly test a list of valid integers. But we  can write our tests as follows instead of using a setup method:

```python
import pytest
from stats import StatsList


@pytest.fixture
def valid_stats():
    return StatsList([1, 2, 2, 3, 3, 4])


def test_mean(valid_stats):
    assert valid_stats.mean() == 2.5


def test_median(valid_stats):
    assert valid_stats.median() == 2.5
    valid_stats.append(4)
    assert valid_stats.median() == 3


def test_mode(valid_stats):
    assert valid_stats.mode() == [2, 3]
    valid_stats.remove(2)
    assert valid_stats.mode() == [3]
```

Each of the three test methods accepts a parameter named `valid_stats`; this parameter is created by calling the `valid_stats` function, which was decorated with `@pytest.fixture`.

Fixtures can do a lot more than return basic variables. A `request` object can be passed into the fixture factory to provide extremely useful methods and attributes to modify the funcarg's behavior. The `module`, `cls`, and `function` attributes allow us to see exactly which test is requesting the fixture. The `config` attribute allows us to check command-line arguments and a great deal of other configuration data.

If  we implement the fixture as a generator, we can run cleanup code after  each test is run. This provides the equivalent of a teardown method,  except on a per-fixture basis. We can use it to clean up files, close  connections, empty lists, or reset queues. For example, the following  code tests the `os.mkdir` functionality by creating a temporary directory fixture:

```python
import pytest
import tempfile
import shutil
import os.path


@pytest.fixture
def temp_dir(request):
    dir = tempfile.mkdtemp()
    print(dir)
    yield dir
    shutil.rmtree(dir)


def test_osfiles(temp_dir):
    os.mkdir(os.path.join(temp_dir, "a"))
    os.mkdir(os.path.join(temp_dir, "b"))
    dir_contents = os.listdir(temp_dir)
    assert len(dir_contents) == 2
    assert "a" in dir_contents
    assert "b" in dir_contents
```

The fixture creates a  new empty temporary directory for files to be created in. It yields  this for use in the test, but removes that directory (using `shutil.rmtree`, which recursively removes a directory and anything inside it) after the test has completed. The filesystem is then left in the same state in which it started.

We can pass a `scope`  parameter to create a fixture that lasts longer than one test. This is  useful when setting up an expensive operation that can be reused by  multiple tests, as long as the resource reuse doesn't break the atomic  or unit nature of the tests (so that one test does not rely on, and is  not impacted by, a previous one). For example, if we were  to test the following echo server, we may want to run only one instance  of the server in a separate process, and then have multiple tests  connect to that instance:

```python
import socket 
 
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM) 
s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1) 
s.bind(('localhost',1028)) 
s.listen(1) 
 
    while True: 
        client, address = s.accept() 
        data = client.recv(1024) 
        client.send(data) 
        client.close() 
```

All this code does is  listen on a specific port and wait for input from a client socket. When  it receives input, it sends the same value back. To test this, we can  start the server in a separate process and cache the result for use in  multiple tests. Here's how the test code might look:

```python
import subprocess
import socket
import time
import pytest


@pytest.fixture(scope="session")
def echoserver():
    print("loading server")
    p = subprocess.Popen(["python3", "echo_server.py"])
    time.sleep(1)
    yield p
    p.terminate()


@pytest.fixture
def clientsocket(request):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(("localhost", 1028))
    yield s
    s.close()


def test_echo(echoserver, clientsocket):
    clientsocket.send(b"abc")
    assert clientsocket.recv(3) == b"abc"


def test_echo2(echoserver, clientsocket):
    clientsocket.send(b"def")
    assert clientsocket.recv(3) == b"def"
```

We've  created two fixtures here. The first runs the echo server in a separate  process, and yields the process object, cleaning it up when it's  finished. The second instantiates a new socket object for each test, and closes the socket when the test has completed.

The first fixture is the one we're currently interested in. From the `scope="session"` keyword argument passed into the decorator's constructor, `pytest` knows that we only want this fixture to be initialized and terminated once for the duration of the unit test session.

The scope can be one of the strings`class`,`module`, `package`,  or`session`. It determines just how long the argument will be cached. We set it to`session` in this example, so it is cached for the duration of the entire`pytest`run. The process will not be terminated or restarted until all tests have run. The`module` scope, of course, caches it only for tests in that module, and the`class` scope treats the object more like a normal class setup and teardown.

info> At the time the third edition of this book went to print, the `package` scope was labeled experimental in `pytest`. Be careful with it, and they request that you supply bug reports.

# Skipping Tests with pytest

As with the `unittest` module, it is frequently necessary to skip tests in `pytest`,  for a similar variety of reasons: the code being tested hasn't been  written yet, the test only runs on certain interpreters or operating  systems, or the test is time-consuming and should only be run under  certain circumstances.

We can skip tests at any point in our code, using the `pytest.skip`  function. It accepts a single argument: a string describing why it has  been skipped. This function can be called anywhere. If we call it inside  a test function, the test will be skipped. If we call it at the module  level, all the tests in that module will be skipped. If we call it  inside a fixture, all tests that call that funcarg will be skipped.

Of  course, in all these locations, it is often desirable to skip tests  only if certain conditions are or are not met. Since we can execute the `skip` function at any place in Python code, we can execute it inside an `if` statement. So we may write a test that looks as follows:

```python
import sys 
import pytest 
 
def test_simple_skip(): 
    if sys.platform != "fakeos": 
        pytest.skip("Test works only on fakeOS") 
     
    fakeos.do_something_fake() 
    assert fakeos.did_not_happen 
```

That's some pretty silly code, really. There is no Python platform named `fakeos`, so this test will skip on all operating systems. It shows how we can skip conditionally, and since the `if` statement can check any valid conditional, we have a lot of power over when tests are skipped. Often, we check `sys.version_info` to check the Python interpreter version, `sys.platform` to check the operating system, or `some_library.__version__` to check whether we have a recent enough version of a given API.

Since  skipping an individual test method or function based on a certain  conditional is one of the most common uses of test skipping, `pytest`  provides a convenience decorator that allows us to do this in one line.  The decorator accepts a single string, which can contain any executable  Python code that evaluates to a Boolean value. For example, the  following test will only run on Python 3 or higher:

```python
@pytest.mark.skipif("sys.version_info <= (3,0)") 
def test_python3(): 
    assert b"hello".decode() == "hello" 
```

The `pytest.mark.xfail` decorator behaves similarly, except that it marks a test as expected to fail, similar to `unittest.expectedFailure()`.  If the test is successful, it will be recorded as a failure. If it  fails, it will be reported as expected behavior. In the case of `xfail`, the conditional argument is optional. If it is not supplied, the test will be marked as expected to fail under all conditions.

The `pytest`  has a ton of other features besides those described here and the  developers are constantly adding innovative new ways to make your  testing experience more enjoyable. They have thorough documentation on their website [here](<https://docs.pytest.org/>).

info> The `pytest` can find and run tests defined using the standard `unittest` library in addition to its own testing infrastructure. This means that if you want to migrate from `unittest` to `pytest`, you don't have to rewrite all your old tests.