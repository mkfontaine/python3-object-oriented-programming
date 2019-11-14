Sometimes, we want to test code that requires an object be supplied  that is either expensive or difficult to construct. In some cases, this  may mean your API needs rethinking to have a more testable interface  (which typically means a more usable interface). But we sometimes find  ourselves writing test code that has a ton of boilerplate to set up  objects that are only incidentally related to the code under test.

For example, imagine we have some code that keeps track of flight statuses in an external key-value store (such as `redis` or `memcache`), such that we can store the timestamp and the most recent status. A basic version of such code might look as follows:

```python
import datetime
import redis


class FlightStatusTracker:
    ALLOWED_STATUSES = {"CANCELLED", "DELAYED", "ON TIME"}

    def __init__(self):
        self.redis = redis.StrictRedis()

    def change_status(self, flight, status):
        status = status.upper()
        if status not in self.ALLOWED_STATUSES:
            raise ValueError("{} is not a valid status".format(status))

        key = "flightno:{}".format(flight)
        value = "{}|{}".format(
            datetime.datetime.now().isoformat(), status
        )
        self.redis.set(key, value)
```

There are a lot of things we ought to test for that `change_status` method. We should check that it raises the appropriate error if a bad status is passed in. We need to ensure that it converts statuses to uppercase. We can see that the key and value have the correct formatting when the `set()` method is called on the `redis` object.

One thing we don't have to check in our unit tests, however, is that the `redis`  object is properly storing the data. This is something that absolutely  should be tested in integration or application testing, but at the unit  test level, we can assume that the py-redis developers have tested their  code and that this method does what we want it to. As a rule, unit  tests should be self-contained and shouldn't rely on the existence of outside resources, such as a running Redis instance.

Instead, we only need to test that the `set()` method was called the appropriate number of times and with the appropriate arguments. We can use `Mock()`  objects in our tests to replace the troublesome method with an object  we can introspect. The following example illustrates the use of `Mock`:

```python
from flight_status_redis import FlightStatusTracker
from unittest.mock import Mock
import pytest


@pytest.fixture
def tracker():
    return FlightStatusTracker()


def test_mock_method(tracker):
    tracker.redis.set = Mock()
    with pytest.raises(ValueError) as ex:
        tracker.change_status("AC101", "lost")
    assert ex.value.args[0] == "LOST is not a valid status"
    assert tracker.redis.set.call_count == 0
```

This test, written using `pytest` syntax, asserts that the correct exception is raised when an inappropriate argument is passed in. In addition, it creates a `Mock` object for the `set` method and makes sure that it is never called. If it was, it would mean there was a bug in our exception handling code.

Simply  replacing the method worked fine in this case, since the object being  replaced was destroyed in the end. However, we often want to replace a  function or method only for the duration of a test. For example, if we  want to test the timestamp formatting in the `Mock` method, we need to know exactly what `datetime.datetime.now()`  is going to return. However, this value changes from run to run. We  need some way to pin it to a specific value so we can test it  deterministically.

Temporarily setting a library function to a  specific value is one of the few valid use cases for monkey-patching.  The mock library provides a patch context manager that allows us to  replace attributes on existing libraries with mock objects. When the  context manager exits, the original attribute is automatically restored  so as not to impact other test cases. Here's an example:

```python
import datetime
from unittest.mock import patch

def test_patch(tracker):
    tracker.redis.set = Mock()
    fake_now = datetime.datetime(2015, 4, 1)
    with patch("datetime.datetime") as dt:
        dt.now.return_value = fake_now
        tracker.change_status("AC102", "on time")
    dt.now.assert_called_once_with()
    tracker.redis.set.assert_called_once_with(
        "flightno:AC102", "2015-04-01T00:00:00|ON TIME"
    )
```

In the preceding example, we first construct a value called `fake_now`, which we will set as the return value of the `datetime.datetime.now` function. We have to construct this object before we patch `datetime.datetime`, because otherwise we'd be calling the patched `now` function before we constructed it.

The `with` statement invites the patch to replace the `datetime.datetime` module with a mock object, which is returned as the `dt `value. The neat thing about mock objects is that any time you access an attribute or method on that object, it returns another mock object. Thus, when we access `dt.now`, it gives us a new mock object. We set the `return_value` of that object to our `fake_now` object. Now, whenever the `datetime.datetime.now`  function is called, it will return our object instead of a new mock  object. But when the interpreter exits the context manager, the original  `datetime.datetime.now()` functionality is restored.

After calling our `change_status` method with known values, we use the `assert_called_once_with` function of the `Mock` class to ensure that the `now` function was indeed called exactly once with no arguments. We then call it a second time to prove that the `redis.set` method was called with arguments that were formatted as we expected them to be.

info> Mocking  dates so you can have deterministic test results is a common patching  scenario. If you are in a situation where you are doing a lot of this,  you might appreciate the `freezegun` and `pytest-freezegun` projects available in the Python Package Index.

The previous example is a good indication of how writing tests can guide our API design. The `FlightStatusTracker` object looks sensible at first glance; we construct a `redis`  connection when the object is constructed, and we call into it when we  need it. When we write tests for this code, however, we discover that  even if we mock out that `self.redis` variable on a `FlightStatusTracker`, the `redis` connection still has to be constructed. This call actually fails if there is no Redis server running, and our tests also fail.

We could solve this problem by mocking out the `redis.StrictRedis` class to return a mock in a `setUp` method. A better idea, however, might be to rethink our implementation. Instead of constructing the `redis` instance inside`__init__`, perhaps we should allow the user to pass one in, as in the following example:

```python
    def __init__(self, redis_instance=None): 
        self.redis = redis_instance if redis_instance else redis.StrictRedis() 
```

This allows us to pass a mock in when we are testing, so the `StrictRedis` method never gets constructed. Additionally, it allows any client code that talks to `FlightStatusTracker` to pass in their own `redis`  instance. There are a variety of reasons they might want to do this:  they may have already constructed one for other parts of their code;  they may have created an optimized implementation of the `redis`  API; perhaps they have one that logs metrics to their internal  monitoring systems. By writing a unit test, we've uncovered a use case  that makes our API more flexible from the start, rather than waiting for  clients to demand we support their exotic needs.

This has been a brief introduction to the wonders of mocking code. Mocks are part of the standard `unittest` library since Python 3.3, but as you see from these examples, they can also be used with `pytest`  and other libraries. Mocks have other more advanced features that you  may need to take advantage of as your code gets more complicated. For  example, you can use the `spec` argument to  invite a mock to imitate an existing class so that it raises an error if  code tries to access an attribute that does not exist on the imitated  class. You can also construct mock methods that return different  arguments each time they are called by passing a list as the `side_effect` argument. The `side_effect`  parameter is quite versatile; you can also use it to execute arbitrary  functions when the mock is called or to raise an exception.

In  general, we should be quite stingy with mocks. If we find ourselves  mocking out multiple elements in a given unit test, we may end up  testing the mock framework rather than our real code. This serves no  useful purpose whatsoever; after all, mocks are well-tested already! If  our code is doing a lot of this, it's probably another sign that the API  we are testing is poorly designed. Mocks should exist at the boundaries  between the code under test and the libraries they interface with. If this isn't happening, we may need to change the API so that the boundaries are redrawn in a different place.