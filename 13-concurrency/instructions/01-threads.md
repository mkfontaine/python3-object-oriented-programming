Most often, concurrency is created  so that work can continue happening while the program is waiting for  I/O to happen. For example, a server can start processing a new network  request while it waits for data from a previous request to arrive. Or an  interactive program might render an animation or perform a calculation  while waiting for the user to press a key. Bear in mind that while a  person can type more than 500 characters per minute, a computer can  perform billions of instructions per second. Thus, a ton of processing  can happen between individual key presses, even when typing quickly.

It's  theoretically possible to manage all this switching between activities  within your program, but it would be virtually impossible to get right.  Instead, we can rely on Python and the operating system to take care of  the tricky switching part, while we create objects that appear to be  running independently, but simultaneously. These objects are called **threads**. Let's take a look at a basic example:

```python
from threading import Thread


class InputReader(Thread):
    def run(self):
        self.line_of_text = input()


print("Enter some text and press enter: ")
thread = InputReader()
thread.start()

count = result = 1
while thread.is_alive():
    result = count * count
    count += 1

print("calculated squares up to {0} * {0} = {1}".format(count, result))
print("while you typed '{}'".format(thread.line_of_text))
```

This  example runs two threads. Can you see them? Every program has (at  least) one thread, called the main thread. The code that executes from startup is happening in this thread. The more visible second thread exists as the `InputReader` class.

To construct a thread, we must extend the `Thread` class and implement the `run` method. Any code executed by the `run` method happens in a separate thread.

The new thread doesn't start running until we call the `start()`  method on the object. In this case, the thread immediately pauses to  wait for input from the keyboard. In the meantime, the original thread  continues executing from the point `start` was called. It starts calculating squares inside a `while` loop. The condition in the `while` loop checks whether the `InputReader` thread has exited its `run` method yet; once it does, it outputs some summary information to the screen.

If we run the example and type the string `hello world`, the output looks as follows:

```python
Enter some text and press enter:
hello world
calculated squares up to 2448265 * 2448265 = 5993996613696
```

You  will, of course, calculate more or less squares while typing the string  as the numbers are related to both our relative typing speeds, and to  the processor speeds of the computers we are running. When I updated  this example between the first and third edition, my newer system was  able to calculate more than twice as many squares.

A thread only starts running in concurrent mode when we call the `start` method. If we want to take out the concurrent call to see how it compares, we can call `thread.run()` in the place that we originally called `thread.start()`. As shown here, the output is telling:

```markdown
Enter some text and press enter:hello worldcalculated squares up to 1 * 1 = 1while you typed 'hello world'
```

In this case, there is no second thread and the `while` loop never executes. We wasted a lot of CPU power sitting idle while we were typing.

There  are a lot of different patterns for using threads effectively. We won't  be covering all of them, but we will look at a common one so we can  learn about the `join` method. Let's check the current temperature in the capital city of each province and territory in Canada:

```python
from threading import Thread
import time
from urllib.request import urlopen
from xml.etree import ElementTree


CITIES = {
    "Charlottetown": ("PE", "s0000583"),
    "Edmonton": ("AB", "s0000045"),
    "Fredericton": ("NB", "s0000250"),
    "Halifax": ("NS", "s0000318"),
    "Iqaluit": ("NU", "s0000394"),
    "Québec City": ("QC", "s0000620"),
    "Regina": ("SK", "s0000788"),
    "St. John's": ("NL", "s0000280"),
    "Toronto": ("ON", "s0000458"),
    "Victoria": ("BC", "s0000775"),
    "Whitehorse": ("YT", "s0000825"),
    "Winnipeg": ("MB", "s0000193"),
    "Yellowknife": ("NT", "s0000366"),
}


class TempGetter(Thread):
    def __init__(self, city):
        super().__init__()
        self.city = city
        self.province, self.code = CITIES[self.city]

    def run(self):
        url = (
            "http://dd.weatheroffice.ec.gc.ca/citypage_weather/xml/"
            f"{self.province}/{self.code}_e.xml"
        )
        with urlopen(url) as stream:
            xml = ElementTree.parse(stream)
            self.temperature = xml.find(
                "currentConditions/temperature"
            ).text


threads = [TempGetter(c) for c in CITIES]
start = time.time()
for thread in threads:
    thread.start()

for thread in threads:
    thread.join()

for thread in threads:
    print(f"it is {thread.temperature}°C in {thread.city}")
print(
    "Got {} temps in {} seconds".format(
        len(threads), time.time() - start
    )
)
```

This code constructs 10 threads before starting them. Notice how we can override the constructor to pass them into the `Thread` object, remembering to call `super` to ensure the `Thread` is properly initialized.

Data we construct in one thread is accessible from other running threads. The references to global variables inside the `run` method illustrate this. Less obviously, the data passed into the constructor is being assigned to `self`**in the main thread**, but is accessed inside the second thread. This can trip people up; just because a method is on a `Thread` instance does not mean it is magically executed inside that thread.

After the 10 threads have been started, we loop over them again, calling the `join()` method on each. This method says **wait for the thread to complete before doing anything**. We call this ten times in sequence; this `for` loop won't exit until all ten threads have completed.

At  this point, we can print the temperature that was stored on each thread  object. Notice, once again, that we can access data that was  constructed within the thread from the main thread. In threads, all  state is shared by default.

Executing the preceding code on my 100 megabit connection takes about three tenths of a second, and we get the following output:

```markdown
it is 18.5°C in Charlottetown
it is 1.6°C in Edmonton
it is 16.6°C in Fredericton
it is 18.0°C in Halifax
it is -2.4°C in Iqaluit
it is 18.4°C in Québec City
it is 7.4°C in Regina
it is 11.8°C in St. John's
it is 20.4°C in Toronto
it is 9.2°C in Victoria
it is -5.1°C in Whitehorse
it is 5.1°C in Winnipeg
it is 1.6°C in Yellowknife
Got 13 temps in 0.29401135444641113 seconds
```

I'm writing in September, but it's already below freezing up north! If I run this code in a single thread (by changing the `start()` call to `run()` and commenting out the `join()`  loop), it takes closer to four seconds because each 0.3-second request  has to complete before the next one begins. This order of magnitude  speedup shows just how useful concurrent programming can be.

# The Many Problems with Threads

Threads can be useful, especially in other programming languages, but modern Python programmers tend to avoid them for several reasons. As we'll see, there are other ways to code  concurrent programming that are receiving more attention from the  Python community. Let's discuss some of these pitfalls before moving on  to them.

## Shared Memory

The main problem with threads is also their primary  advantage. Threads have access to all the program's memory and thus all  the variables. This can too easily cause inconsistencies in the program  state.

Have you ever encountered a room where a single light has  two switches and two different people turn them on at the same time?  Each person (thread) expects their action to turn the lamp (a variable)  on, but the resulting value (the lamp) is off, which is inconsistent  with those expectations. Now imagine if those two threads were  transferring funds between bank accounts or managing the cruise control for a vehicle.

The solution to this problem in threaded programming is to *synchronize* access  to any code that reads or (especially) writes a shared variable. There  are a few different ways to do this, but we won't go into them here so  we can focus on more Pythonic constructs.

The synchronization  solution works, but it is way too easy to forget to apply it. Worse,  bugs due to inappropriate use of synchronization are really hard to  track down because the order in which threads perform operations is  inconsistent. We can't easily reproduce the error. Usually, it is safest  to force communication between threads to happen using a lightweight  data structure that already uses locks appropriately. Python offers the `queue.Queue` class to do this; its functionality is basically the same as `multiprocessing.Queue`, which we will discuss in the next section.

In  some cases, these disadvantages might be outweighed by the one  advantage of allowing shared memory: it's fast. If multiple threads need  access to a huge data structure, shared memory can provide that access  quickly. However, this advantage is usually nullified by the fact that,  in Python, it is impossible for two threads running on different CPU  cores to be performing calculations at exactly the same time. This brings us to our second problem with threads.

## The Global Interpreter Lock

In order to efficiently manage memory, garbage collection, and calls to machine code in native libraries, Python has a utility called the **global interpreter lock**, or **GIL**.  It's impossible to turn off, and it means that threads are useless in  Python for one thing that they excel at in other languages: parallel  processing. The GIL's primary effect, for our purposes, is to prevent  any two threads from doing work at the exact same time, even if they  have work to do. In this case, **doing work** means  using the CPU, so it's perfectly okay for multiple threads to access  the disk or network; the GIL is released as soon as the thread starts to  wait for something. This is why the weather example worked.

The  GIL is highly disparaged, mostly by people who don't understand what it  is or all the benefits it brings to Python. It would definitely be nice  if our language didn't have this restriction, but the Python development  team have determined that it brings more value than it costs. It makes  the reference implementation easier to maintain and develop, and during  the single-core processor days when Python was originally developed, it  actually made the interpreter faster. The net result of the GIL,  however, is that it limits the benefits that threads bring us, without  alleviating the costs.

info> While  the GIL is a problem in the reference implementation of Python that  most people use, it has been solved in some of the non-standard  implementations, such as IronPython and Jython. Unfortunately, at the  time of publication, none of these support Python 3.

# Thread Overhead

One final limitation of threads, as compared to the asynchronous system we will be discussing later, is the cost of maintaining each thread.  Each thread takes up a certain amount of memory (both in the Python  process and the operating system kernel) to record the state of that  thread. Switching between the threads also uses a (small) amount of CPU  time. This work happens seamlessly without any extra coding (we just  have to call `start()` and the rest is taken care of), but the work still has to happen somewhere.

This  can be alleviated somewhat by structuring our workload so that threads  can be reused to perform multiple jobs. Python provides a `ThreadPool` feature to handle this. It is shipped as part of the multiprocessing library and behaves identically to `ProcessPool`, which we will discuss shortly, so let's defer that discussion until the next lesson.