The decorator pattern allows us to **wrap** an  object that provides core functionality with other objects that alter  this functionality. Any object that uses the decorated object will  interact with it in exactly the same way as if it were undecorated (that  is, the interface of the decorated object is identical to that of the  core object).

There are two primary uses of the decorator pattern:

- Enhancing the response of a component as it sends data to a second component
- Supporting multiple optional behaviors

The  second option is often a suitable alternative to multiple inheritance.  We can construct a core object, and then create a decorator wrapping  that core. Since the decorator object has the same interface as the core  object, we can even wrap the new object in other decorators. Here's how  it looks in a UML diagram:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/0f28e236-afbe-4e3a-a40d-0ae42202c157.png)

Here, **Core** and all the decorators implement a specific **Interface**. The decorators maintain a reference to another instance of that **Interface**  via composition. When called, the decorator does some added processing  before or after calling its wrapped interface. The wrapped object may be  another decorator, or the core functionality. While multiple decorators  may wrap each other, the object in the **center** of all those decorators provides the core functionality.

# A Decorator Example

Let's look at an example from network programming. We'll be using a TCP socket. The `socket.send()`  method takes a string of input bytes and outputs them to the receiving  socket at the other end. There are plenty of libraries that accept  sockets and access this function to send data on the stream. Let's  create such an object; it will be an interactive shell that waits for a  connection from a client and then prompts the user for a string  response:

```Python
import socket


def respond(client):
    response = input("Enter a value: ")
    client.send(bytes(response, "utf8"))
    client.close()


server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(("localhost", 2401))
server.listen(1)
try:
    while True:
        client, addr = server.accept()
        respond(client)
finally:
    server.close()
```

The `respond` function accepts a `socket`  parameter and prompts for data to be sent as a reply, then sends it. To  use it, we construct a server socket and tell it to listen on port `2401` (I picked the port randomly) on the local computer. When a client connects, it calls the `respond` function, which requests data interactively and responds appropriately. The important thing to notice is that the `respond` function only cares about two methods of the socket interface: `send` and `close`.

To test this, we can write a very simple client that connects to the same port and outputs the response before exiting:

```python
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("localhost", 2401))
print("Received: {0}".format(client.recv(1024)))
client.close()
```

To use these programs, follow these steps:

1. Start the server in one Terminal.
2. Open a second Terminal window and run the client.
3. At the **`Enter a value:`** prompt in the server window, type a value and press **Enter**.
4. The  client will receive what you typed, print it to the console, and exit.  Run the client a second time; the server will prompt for a second value.

The result will look something like this:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/5bed7350-8b6f-4f19-a003-27a6a523080e.png)

Now, looking back at our server code, we see two sections. The `respond` function sends data into a `socket` object. The remaining script is responsible for creating that `socket`  object. We'll create a pair of decorators that customize the socket  behavior without having to extend or modify the socket itself.

Let's start with a **logging** decorator. This object outputs any data being sent to the server's console before it sends it to the client:

```python
class LogSocket:
    def __init__(self, socket):
        self.socket = socket

    def send(self, data):
        print(
            "Sending {0} to {1}".format(
                data, self.socket.getpeername()[0]
            )
        )
        self.socket.send(data)

    def close(self):
        self.socket.close()
```

This class decorates a `socket` object and presents the `send` and `close` interface to client sockets. A better decorator would also implement (and possibly customize) all of the remaining `socket` methods. It should properly implement all of the arguments to `send`, (which actually accepts an optional flags argument) as well, but let's keep our example simple. Whenever `send` is called on this object, it logs the output to the screen before sending data to the client using the original socket.

We only have to change one line in our original code to use this decorator. Instead of calling `respond` with the socket, we call it with a decorated socket:

```python
respond(LogSocket(client)) 
```

While that's quite simple, we have to ask ourselves why we didn't just extend the `socket` class and override the `send` method. We could call `super().send` to do the actual sending, after we logged it. There is nothing wrong with this design either.

When  faced with a choice between decorators and inheritance, we should only  use decorators if we need to modify the object dynamically, according to  some condition. For example, we may only want to enable the logging  decorator if the server is currently in debugging mode. Decorators also  beat multiple inheritance when we have more than one optional behavior.  As an example, we can write a second decorator that compresses data  using `gzip` compression whenever `send` is called:

```python
import gzip
from io import BytesIO


class GzipSocket:
    def __init__(self, socket):
        self.socket = socket

    def send(self, data):
        buf = BytesIO()
        zipfile = gzip.GzipFile(fileobj=buf, mode="w")
        zipfile.write(data)
        zipfile.close()
        self.socket.send(buf.getvalue())

    def close(self):
        self.socket.close()
```

The `send` method in this version compresses the incoming data before sending it on to the client.

Now  that we have these two decorators, we can write code that dynamically  switches between them when responding. This example is not complete, but  it illustrates the logic we might follow to mix and match decorators:

```python
        client, addr = server.accept() 
        if log_send: 
            client = LogSocket(client) 
        if client.getpeername()[0] in compress_hosts: 
            client = GzipSocket(client) 
        respond(client) 
```

This code checks a hypothetical configuration variable named `log_send`. If it's enabled, it wraps the socket in a `LogSocket`  decorator. Similarly, it checks whether the client that has connected  is in a list of addresses known to accept compressed content. If so, it  wraps the client in a `GzipSocket` decorator.  Notice that none, either, or both of the decorators may be enabled,  depending on the configuration and connecting client. Try writing this  using multiple inheritance and see how confused you get!

# Decorators in Python

The decorator pattern is useful in Python, but there are other options. For example, we may be able to use monkey-patching (for example, `socket.socket.send = log_send`) to get a similar effect. Single inheritance, where the **optional** calculations  are done in one large method, could be an option, and multiple  inheritance should not be written off just because it's not suitable for  the specific example seen previously.

In Python, it is very  common to use this pattern on functions. As we saw in a previous  chapter, functions are objects too. In fact, function decoration is so  common that Python provides a special syntax to make it easy to apply  such decorators to functions.

For example, we can look at the  logging example in a more general way. Instead of logging, only send  calls on sockets; we may find it helpful to log all calls to certain  functions or methods. The following example implements a decorator that  does just this:

```python
import time


def log_calls(func):
    def wrapper(*args, **kwargs):
        now = time.time()
        print(
            "Calling {0} with {1} and {2}".format(
                func.__name__, args, kwargs
            )
        )
        return_value = func(*args, **kwargs)
        print(
            "Executed {0} in {1}ms".format(
                func.__name__, time.time() - now
            )
        )
        return return_value

    return wrapper


def test1(a, b, c):
    print("\ttest1 called")


def test2(a, b):
    print("\ttest2 called")


def test3(a, b):
    print("\ttest3 called")
    time.sleep(1)


test1 = log_calls(test1)
test2 = log_calls(test2)
test3 = log_calls(test3)

test1(1, 2, 3)
test2(4, b=5)
test3(6, 7)
```

This decorator function is very similar to the  example we explored earlier; in those cases, the decorator took a  socket-like object and created a socket-like object. This time, our  decorator takes a function object and returns a new function object.  This code comprises three separate tasks:

- A function, `log_calls`, that accepts another function
- This function defines (internally) a new function, named `wrapper`, that does some extra work before calling the original function
- The inner function is returned from the outer function

Three sample functions demonstrate the decorator in use. The third one includes a `sleep`  call to demonstrate the timing test. We pass each function into the  decorator, which returns a new function. We assign this new function to  the original variable name, effectively replacing the original function  with a decorated one.

This syntax allows us to build up decorated  function objects dynamically, just as we did with the socket example. If  we don't replace the name, we can even keep decorated and non-decorated  versions for different situations.

Typically, these decorators  are general modifications that are applied permanently to different  functions. In this situation, Python supports a special syntax to apply  the decorator at the time the function is defined. We've already seen  this syntax in a few places; now, let's understand how it works.

Instead of applying the decorator function after the method definition, we can use the `@decorator` syntax to do it all at once:

```python
@log_calls 
def test1(a,b,c): 
    print("\ttest1 called") 
```

The primary benefit of  this syntax is that we can easily see that the function has been  decorated whenever we read the function definition. If the decorator is  applied later, someone reading the code may miss that the function has  been altered at all. Answering a question like, **Why is my program logging function calls to the console?** can  become much more difficult! However, the syntax can only be applied to  functions we define, since we don't have access to the source code of  other modules. If we need to decorate functions that are part of  somebody else's third-party library, we have to use the earlier syntax.

There  is more to the decorator syntax than we've seen here. We don't have  room to cover the advanced topics here, so check the Python reference manual or other tutorials for more  information. Decorators can be created as callable objects, not just  functions that return functions. Classes can also be decorated; in that  case, the decorator returns a new class instead of a new function.  Finally, decorators can take arguments to customize them on a  per-function basis.