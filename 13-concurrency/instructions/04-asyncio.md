AsyncIO is the current state of the art in Python concurrent programming. It combines the concept of futures and an event loop with the coroutines we discussed in **The Iterator Pattern**.  The result is about as elegant and easy to understand as it is possible  to get when writing concurrent code, though that isn't saying a lot!

AsyncIO  can be used for a few different concurrent tasks, but it was  specifically designed for network I/O. Most networking applications,  especially on the server side, spend a lot of time waiting for data to  come in from the network. This can be solved by handling each client in a  separate thread, but threads use up memory and other resources. AsyncIO  uses coroutines as a sort of lightweight thread.

The library provides its own event loop, obviating the need for the several lines long the `while` loop in the previous example. However, event loops come with a cost. When we run code in an `async` task on the event loop, that code **must**  return immediately, blocking neither on I/O nor on long-running  calculations. This is a minor thing when writing our own code, but it  means that any standard library or third-party functions that block on  I/O have to have non-blocking versions created.

AsyncIO solves this by creating a set of coroutines that use `async` and `await` syntax to return control to the event loop immediately when code will block. These keywords replace the `yield`, `yield from`, and `send` syntax we used in the raw coroutines we saw earlier, as well as the need to manually advance to the first *send* location.  The result is concurrent code that we can reason about as if it were  sequential. The event loop takes care of checking whether the blocking  call has completed and performing any subsequent tasks, much as we did manually in the previous section.

# AsyncIO in Action

A canonical example of a blocking function is the `time.sleep` call. Let's use the asynchronous version of this call to illustrate the basics of an AsyncIO event loop, as follows:

```python
import asyncio
import random


async def random_sleep(counter):
    delay = random.random() * 5
    print("{} sleeps for {:.2f} seconds".format(counter, delay))
    await asyncio.sleep(delay)
    print("{} awakens".format(counter))


async def five_sleepers():
    print("Creating five tasks")
    tasks = [asyncio.create_task(random_sleep(i)) for i in range(5)]
    print("Sleeping after starting five tasks")
    await asyncio.sleep(2)
    print("Waking and waiting for five tasks")
    await asyncio.gather(*tasks)


asyncio.get_event_loop().run_until_complete(five_sleepers())
print("Done five tasks")
```

This is a fairly basic  example, but it covers several features of AsyncIO programming. It is  easiest to understand in the order that it executes, which is more or  less bottom to top.

Here's how one execution of the script looks:

```markdown
Creating five tasks
Sleeping after starting five tasks
0 sleeps for 3.42 seconds
1 sleeps for 4.16 seconds
2 sleeps for 0.75 seconds
3 sleeps for 3.55 seconds
4 sleeps for 0.39 seconds
4 awakens
2 awakens
Waking and waiting for five tasks
0 awakens
3 awakens
1 awakens
Done five tasks
```

The second to last line gets the event loop and instructs it to run a task until it is finished. The task in question is named `five_sleepers`.  Once that task has done its work, the loop will exit and our code will  terminate. As asynchronous programmers, we don't need to know too much  about what happens inside that `run_until_complete`  call, but be aware that a lot is going on. It's a souped-up coroutine  version of the futures loop we wrote in the previous lesson, which  knows how to deal with iteration, exceptions, function returns, parallel  calls, and more.

A task, in this context, is an object that `asyncio` knows how to schedule on the event loop. This includes the following:

- Coroutines defined with the `async` and `await` syntax.
- Coroutines decorated with `@asyncio.coroutine` and using the `yield from` syntax (this is an older model, deprecated in favor of `async` and `await`).
- `asyncio.Future` objects. These are almost identical to the `concurrent.futures` you saw in the previous section, but for use with `asyncio`.
- Any awaitable object, that is, one with an `__await__` function.

In this example, all the tasks are coroutines; we'll see some of the others in later examples.

Look a little more closely at that `five_sleepers` future. The coroutine first constructs five instances of the `random_sleep` coroutine. These are each wrapped in a `asyncio.create_task`  call, which adds the future to the loop's task queue so they can  execute and start immediately when control is returned to the loop.

That control is returned whenever we call `await`. In this case, we call `await asyncio.sleep`  to pause the execution of the coroutine for two seconds. During the  break, the event loop executes the tasks that it has queued up: namely,  the five `random_sleep` tasks.

When the sleep call in the `five_sleepers` task wakes up, it calls `asyncio.gather`.  This function accepts tasks as varargs, and awaits each of them (among  other things, to keep the loop running safely) before returning.

Each of the `random_sleep` coroutines prints a starting message, then sends control back to the event loop for a specific amount of time using its own `await` calls. When the sleep has completed, the event loop passes control back to the relevant `random_sleep` task, which prints its awakening message before returning.

Note that any tasks that take less than two seconds to complete will output their own awakening messages before the original `five_sleepers` coroutine awakes to run until the `gather`  task is called. Since the event queue is now empty (all six coroutines  have run to completion and are not awaiting any tasks), the `run_until_complete` call is able to terminate and the program ends.

The `async` keyword acts as documentation notifying the python interpreter (and coder) that the coroutine contains the `await`  calls. It also does some work to prepare the coroutine to run on the  event loop. It behaves much like a decorator; in fact, back in Python  3.4, this was implemented as an `@asyncio.coroutine` decorator.

# Reading an AsyncIO Future

An AsyncIO coroutine executes each line in order until it encounters an `await`  statement, at which point, it returns control to the event loop. The  event loop then executes any other tasks that are ready to run,  including the one that the original coroutine was waiting on. Whenever  that child task completes, the event loop sends the result back into the  coroutine so that it can pick up execution until it encounters another `await` statement or returns.

This  allows us to write code that executes synchronously until we explicitly  need to wait for something. As a result, there is no nondeterministic  behavior of threads, so we don't need to worry nearly so much about  shared state.

info> It's  still a good idea to avoid accessing shared state from inside a  coroutine. It makes your code much easier to reason about. More  importantly, even though an ideal world might have all asynchronous  execution happening inside coroutines, the reality is that some futures  are executed behind the scenes inside threads or processes. Stick to a **share nothing** philosophy to avoid a ton of difficult bugs.

In  addition, AsyncIO allows us to collect logical sections of code  together inside a single coroutine, even if we are waiting for other  work elsewhere. As a specific instance, even though the `await asyncio.sleep` call in the `random_sleep`  coroutine is allowing a ton of stuff to happen inside the event loop,  the coroutine itself looks like it's doing everything in order. This  ability to read related pieces of asynchronous code without worrying  about the machinery that waits for tasks to complete is the primary  benefit of the AsyncIO module.

# AsyncIO for Networking

AsyncIO was specifically designed  for use with network sockets, so let's implement a DNS server. More  accurately, let's implement one extremely basic feature of a DNS server.

The DNS's basic purpose is to translate domain names, such as [https://www.python.org/](<https://www.python.org/>), into IP addresses, such as IPv4 addresses (for example `23.253.135.79`) or IPv6 addresses (such as `2001:4802:7901:0:e60a:1375:0:6`).  It has to be able to perform many types of queries and know how to  contact other DNS servers if it doesn't have the answer required. We  won't be implementing any of this, but the following example is able to  respond directly to a standard DNS query to look up IPs for a few sites:

```python
import asyncio
from contextlib import suppress

ip_map = {
    b"facebook.com.": "173.252.120.6",
    b"yougov.com.": "213.52.133.246",
    b"wipo.int.": "193.5.93.80",
    b"dataquest.io.": "104.20.20.199",
}


def lookup_dns(data):
    domain = b""
    pointer, part_length = 13, data[12]
    while part_length:
        domain += data[pointer : pointer + part_length] + b"."
        pointer += part_length + 1
        part_length = data[pointer - 1]

    ip = ip_map.get(domain, "127.0.0.1")

    return domain, ip


def create_response(data, ip):
    ba = bytearray
    packet = ba(data[:2]) + ba([129, 128]) + data[4:6] * 2
    packet += ba(4) + data[12:]
    packet += ba([192, 12, 0, 1, 0, 1, 0, 0, 0, 60, 0, 4])
    for x in ip.split("."):
        packet.append(int(x))
    return packet


class DNSProtocol(asyncio.DatagramProtocol):
    def connection_made(self, transport):
        self.transport = transport

    def datagram_received(self, data, addr):
        print("Received request from {}".format(addr[0]))
        domain, ip = lookup_dns(data)
        print(
            "Sending IP {} for {} to {}".format(
                domain.decode(), ip, addr[0]
            )
        )
        self.transport.sendto(create_response(data, ip), addr)


loop = asyncio.get_event_loop()
transport, protocol = loop.run_until_complete(
    loop.create_datagram_endpoint(
        DNSProtocol, local_addr=("127.0.0.1", 4343)
    )
)
print("DNS Server running")

with suppress(KeyboardInterrupt):
    loop.run_forever()
transport.close()
loop.close()
```

This example sets up a dictionary  that dumbly maps a few domains to IPv4 addresses. It is followed by two  functions that extract information from a binary DNS query packet and  construct the response. We won't be discussing these; if you want to  know more about DNS read RFC (**request for comment**, the format for defining most IPs) `1034` and `1035`.

You can test this service by running the following command in another terminal:

```bash
nslookup -port=4343 facebook.com localhost
```

Let's  get on with the entree. AsyncIO networking revolves around the  intimately linked concepts of transports and protocols. A protocol is a  class that has specific methods that are called when relevant events  happen. Since DNS runs on top of **UDP** (**User Datagram Protocol**), we build our protocol class as a subclass of `DatagramProtocol`.  There are a variety of events this class can respond to. We are  specifically interested in the initial connection occurring (solely so  that we can store the transport for future use) and the `datagram_received` event. For DNS, each received datagram must be parsed and responded to, at which point, the interaction is over.

So,  when a datagram is received, we process the packet, look up the IP, and  construct a response using the functions we aren't talking about  (they're black sheep in the family). Then, we instruct the underlying  transport to send the resulting packet back to the requesting client  using its `sendto` method.

The  transport essentially represents a communication stream. In this case,  it abstracts away all the fuss of sending and receiving data on a UDP  socket on an event loop. There are similar transports for interacting  with TCP sockets and subprocesses, for example.

The UDP transport is constructed by calling the loop's `create_datagram_endpoint`  coroutine. This constructs the appropriate UDP socket and starts  listening on it. We pass it the address that the socket needs to listen  on and, importantly, the protocol class we created so that the transport knows what to call when it receives data.

Since the process of initializing a socket takes a non-trivial amount of time and would block the event loop, the `create_datagram_endpoint`  function is a coroutine. In our example, we don't need to do anything  while we wait for this initialization, so we wrap the call in `loop.run_until_complete`.  The event loop takes care of managing the future, and when it's  complete, it returns a tuple of two values: the newly initialized  transport and the protocol object that was constructed from the class we  passed in.

Behind the scenes, the transport has set up a task on  the event loop that is listening for incoming UDP connections. All we  have to do, then, is start the event loop running with the call to `loop.run_forever()`  so that the task can process these packets. When the packets arrive,  they are processed on the protocol and everything just works.

The  only other major thing to pay attention to is that transports (and,  indeed, event loops) are supposed to be closed when we are finished with  them. In this case, the code runs just fine without the two calls to `close()`,  but if we were constructing transports on the fly (or just doing proper  error handling!), we'd need to be quite a bit more conscious of it.

You may have been dismayed to see how much boilerplate is required in setting  up a protocol class and the underlying transport. AsyncIO provides an  abstraction on top of these two key concepts, called streams. We'll see  an example of streams in the TCP server in the next example.

# Using Executors to Wrap Blocking Code

AsyncIO provides its own version of the futures  library to allow us to run code in a separate thread or process when  there isn't an appropriate non-blocking call to be made. This allows us  to combine threads and processes with the asynchronous model. One of the  more useful applications of this feature is to get the best of both  worlds when an application has bursts of I/O-bound and CPU-bound  activity. The I/O-bound portions can happen in the event loop, while the  CPU-intensive work can be spun off to a different process. To  illustrate this, let's implement **sorting as a service** using AsyncIO:

```python
import asyncio
import json
from concurrent.futures import ProcessPoolExecutor


def sort_in_process(data):
    nums = json.loads(data.decode())
    curr = 1
    while curr < len(nums):
        if nums[curr] >= nums[curr - 1]:
            curr += 1
        else:
            nums[curr], nums[curr - 1] = nums[curr - 1], nums[curr]
            if curr > 1:
                curr -= 1

    return json.dumps(nums).encode()


async def sort_request(reader, writer):
    print("Received connection")
    length = await reader.read(8)
    data = await reader.readexactly(int.from_bytes(length, "big"))
    result = await asyncio.get_event_loop().run_in_executor(
        None, sort_in_process, data
    )
    print("Sorted list")
    writer.write(result)
    writer.close()
    print("Connection closed")


loop = asyncio.get_event_loop()
loop.set_default_executor(ProcessPoolExecutor())
server = loop.run_until_complete(
    asyncio.start_server(sort_request, "127.0.0.1", 2015)
)
print("Sort Service running")

loop.run_forever()
server.close()
loop.run_until_complete(server.wait_closed())
loop.close()
```

This is an example of good code implementing some really stupid ideas. The whole idea of sorting as a service is pretty ridiculous. Using our own sorting algorithm instead of calling Python's `sorted` is even worse. The algorithm we used is called gnome sort, or in some cases, **stupid sort**.  It is a slow sort algorithm implemented in pure Python. We defined our  own protocol instead of using one of the many perfectly suitable  application protocols that exist in the wild. Even the idea of using  multiprocessing for parallelism might be suspect here; we still end up  passing all the data into and out of the subprocesses. Sometimes, it's  important to take a step back from the program you are writing and ask  yourself whether you are trying to meet the right goals.

But  ignoring the workload, let's look at some of the smart features of this  design. First, we are passing bytes into and out of the subprocess.  This is a lot smarter than decoding  the JSON in the main process. It means the (relatively expensive)  decoding can happen on a different CPU. Also, pickled JSON strings are  generally smaller than pickled lists, so less data is passed between processes.

Second,  the two methods are very linear; it looks like code is being executed  one line after another. Of course, in AsyncIO, this is an illusion, but  we don't have to worry about shared memory or concurrency primitives.

## Streams

The sort service example should look familiar by now, as it has a similar boilerplate to other AsyncIO programs. However, there are a few differences. We called `start_server` instead of `create_server`.  This method hooks into AsyncIO's streams instead of using the  underlying transport/protocol code. It allows us to pass in a normal  coroutine, which receives reader and writer parameters. These both  represent streams of bytes that can be read from and written, like files  or sockets. Second, because this is a TCP server instead of UDP, there  is some socket cleanup required when the program finishes. This cleanup  is a blocking call, so we have to run the `wait_closed` coroutine on the event loop.

Streams are fairly simple to understand. Reading is a potentially blocking call so we have to call it with `await`. Writing doesn't block; it just puts the data in a queue, which AsyncIO sends out in the background.

Our code inside the `sort_request`  method makes two read requests. First, it reads 8 bytes from the wire  and converts them to an integer using big endian notation. This integer  represents the number of bytes of data the client intends to send. So,  in the next call, to `readexactly`, it reads that many bytes. The difference between `read` and `readexactly`  is that the former will read up to the requested number of bytes, while  the latter will buffer reads until it receives all of them, or until  the connection closes.

## Executors

Now let's look at the executor code. We import the exact same `ProcessPoolExecutor` that we used in the previous section. Notice that we don't need a special AsyncIO version of it. The event loop has a handy `run_in_executor` coroutine that we can use to run futures on. By default, the loop runs code in `ThreadPoolExecutor`,  but we can pass in a different executor if we wish. Or, as we did in  this example, we can set a different default when we set up the event  loop by calling `loop.set_default_executor()`.

As  you probably recall, there is not a lot of boilerplate for using  futures with an executor. However, when we use them with AsyncIO, there  is none at all! The coroutine automatically wraps the function call in a  future and submits it to the executor. Our code blocks until the future  completes, while the event loop continues processing other connections,  tasks, or futures. When the future is done, the coroutine wakes up and  continues on to write the data back to the client.

You may be wondering if,  instead of running multiple processes inside an event loop, it might be  better to run multiple event loops in different processes. The answer  is: **maybe**. However, depending on  the exact problem space, we are probably better off running independent  copies of a program with a single event loop than trying to coordinate everything with a master multiprocessing process.

# AsyncIO Clients

Because it is capable of handling many thousands  of simultaneous connections, AsyncIO is very common for implementing  servers. However, it is a generic networking library, and provides full  support for client processes as well. This is pretty important, since  many microservices run servers that act as clients to other servers.

Clients  can be much simpler than servers, as they don't have to be set up to  wait for incoming connections. Like most networking libraries, you just  open a connection, submit your requests, and process any responses. The  main difference is that you need to use `await`  any time you make a potentially blocking call. Here's an example client  for the sort service we implemented in the last lesson:

```python
import asyncio
import random
import json


async def remote_sort():
    reader, writer = await asyncio.open_connection("127.0.0.1", 2015)
    print("Generating random list...")
    numbers = [random.randrange(10000) for r in range(10000)]
    data = json.dumps(numbers).encode()
    print("List Generated, Sending data")
    writer.write(len(data).to_bytes(8, "big"))
    writer.write(data)

    print("Waiting for data...")
    data = await reader.readexactly(len(data))
    print("Received data")
    sorted_values = json.loads(data.decode())
    print(sorted_values)
    print("\n")
    writer.close()


loop = asyncio.get_event_loop()
loop.run_until_complete(remote_sort())
loop.close()
```

We've  hit most of the high points of AsyncIO in this section, and the chapter  has covered many other concurrency primitives. Concurrency is a hard problem  to solve, and no one solution fits all use cases. The most important  part of designing a concurrent system is deciding which of the available  tools is the correct one to use for the problem. We have seen the  advantages and disadvantages of several concurrent systems, and now have  some insight into which are the better choices for different types of  requirements.