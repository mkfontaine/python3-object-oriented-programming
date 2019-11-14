The multiprocessing API was originally designed  to mimic the thread API. However, it has evolved, and in recent  versions of Python 3, it supports more features more robustly. The  multiprocessing library is designed for when CPU-intensive jobs need to  happen in parallel and multiple cores are available (almost all  computers, even a little smartwatch, have multiple cores).  Multiprocessing is not useful when the processes spend a majority of  their time waiting on I/O (for example, network, disk, database, or  keyboard), but it is the way to go for parallel computation.

The  multiprocessing module spins up new operating system processes to do  the work. This means there is an entirely separate copy of the Python  interpreter running for each process. Let's try to parallelize a  compute-heavy operation using similar constructs to those provided by  the `threading` API, as follows:

```python
from multiprocessing import Process, cpu_count
import time
import os


class MuchCPU(Process):
    def run(self):
        print(os.getpid())
        for i in range(200000000):
            pass


if __name__ == "__main__":
    procs = [MuchCPU() for f in range(cpu_count())]
    t = time.time()
    for p in procs:
        p.start()
    for p in procs:
        p.join()
    print("work took {} seconds".format(time.time() - t))
```

This example just ties up the CPU for 200 million iterations. You may not consider this to be useful work, but it can warm up your laptop on a chilly day!

The API should be familiar; we implement a subclass of `Process` (instead of `Thread`) and implement a `run`  method. This method prints out the process ID (a unique number the  operating system assigns to each process on the machine) before doing  some intense (if misguided) work.

Pay special attention to the `if __name__ == '__main__':`  guard around the module level code that prevents it running if the  module is being imported, rather than run as a program. This is good  practice in general, but when using multiprocessing on some operating  systems, it is essential. Behind the scenes, multiprocessing may have to  reimport the module inside the new process in order to execute the `run()`  method. If we allowed the entire module to execute at that point, it  would start creating new processes recursively until the operating  system ran out of resources, crashing your computer.

We construct  one process for each processor core on our machine, then start and join  each of those processes. On my 2017-era 8-core ThinkCenter, the output  looks as follows:

```markdown
25812
25813
25814
25815
25816
25817
25818
25819
work took 6.97506308555603 seconds
```

The first four lines are the process ID that was printed inside each `MuchCPU`  instance. The last line shows that the 200 million iterations can run  in about 13 seconds on my machine. During that 13 seconds, my process  monitor indicated that all four of my cores were running at 100 percent.

If we subclass `threading.Thread` instead of `multiprocessing.Process` in `MuchCPU`, the output looks, as follows:

```markdown
26083
26083
26083
26083
26083
26083
26083
26083
work took 26.710845470428467 seconds
```

This  time, the four threads are running inside the same process and take  over three times as long to run. This is the cost of the GIL; in other  languages, the threaded version would run at least as fast as the  multiprocessing version.

We might expect it to be at least four  times as long, but remember that many other programs are running on my  laptop. In the multiprocessing version, these programs also need a share  of the four CPUs. In the threading version, those programs can use the other seven CPUs instead.

# Multiprocessing Pools

In general, there is no reason to have more processes than there are processors on the computer. There are a few reasons for this:

- Only `cpu_count()` processes can run simultaneously
- Each process consumes resources with a full copy of the Python interpreter
- Communication between processes is expensive
- Creating processes takes a non-zero amount of time

Given these constraints, it makes sense to create at most`cpu_count()`  processes when the program starts and then have them execute tasks as  needed. This has much less overhead than starting a new process for each  task.

It is not difficult to implement a basic series of  communicating processes that does this, but it can be tricky to debug,  test, and get right. Of course, other Python developers have already  done it for us in the form of multiprocessing pools.

Pools  abstract away the overhead of figuring out what code is executing in the  main process and which code is running in the subprocess. The pool  abstraction restricts the number of places in which code in different  processes interacts, making it much easier to keep track of.

Unlike  threads, multiprocessing cannot directly access variables set up by  other threads. Multiprocessing provides a few different ways to implement  interprocess communication. Pools seamlessly hide the process of  passing data between processes. Using a pool looks much like a function  call: you pass data into a function, it is executed in another process  or processes, and when the work is done, a value is returned. It is  important to understand that under the hood, a lot of work is being done  to support this: objects in one process are being pickled and passed  into an operating system process pipe. Then, another process retrieves  data from the pipe and unpickles it. The requested work is done in the  subprocess and a result is produced. The result is pickled and passed back through the pipe. Eventually, the original process unpickles and returns it.

All  this pickling and passing data into pipes takes time and memory.  Therefore, it is ideal to keep the amount and size of data passed into  and returned from the pool to a minimum, and it is only advantageous to  use the pool if a lot of processing has to be done on the data in  question.

info> Pickling  is an expensive operation for even medium-sized Python operations. It  is frequently more expensive to pickle a large object for use in a  separate process than it would be to do the work in the original process  using threads. Make sure you profile your program to ensure the  overhead of multiprocessing is actually worth the overhead of  implementing and maintaining it.

Armed with this knowledge,  the code to make all this machinery work is surprisingly simple. Let's  look at the problem of calculating all the prime factors of a list of  random numbers. This is a common and expensive part of a variety of  cryptography algorithms (not to mention attacks on those algorithms!).  It requires years of processing power to crack the extremely large  numbers used to secure your bank accounts. The following implementation,  while readable, is not at all efficient, but that's okay because we  want to see it using lots of CPU time:

```python
import random
from multiprocessing.pool import Pool


def prime_factor(value):
    factors = []
    for divisor in range(2, value - 1):
        quotient, remainder = divmod(value, divisor)
        if not remainder:
            factors.extend(prime_factor(divisor))
            factors.extend(prime_factor(quotient))
            break
    else:
        factors = [value]
    return factors


if __name__ == "__main__":
    pool = Pool()

    to_factor = [random.randint(100000, 50000000) for i in range(20)]
    results = pool.map(prime_factor, to_factor)
    for value, factors in zip(to_factor, results):
        print("The factors of {} are {}".format(value, factors))
```

Let's focus on the parallel processing  aspects, as the brute force recursive algorithm for calculating factors  is pretty clear. We first construct a multiprocessing pool instance. By  default, this pool creates a separate process for each of the CPU cores  in the machine it is running on.

The `map`  method accepts a function and an iterable. The pool pickles each of the  values in the iterable and passes it into an available process, which  executes the function on it. When that process is finished doing its  work, it pickles the resulting list of factors and passes it back to the  pool. Then, if the pool has more work available, it takes on the next  job.

Once all the pools are finished processing work (which could  take some time), the results list is passed back to the original  process, which has been waiting patiently for all this work to complete.

It is often more useful to use the similar `map_async`  method, which returns immediately even though the processes are still  working. In that case, the results variable would not be a list of  values, but a promise to return a list of values later by calling `results.get()`. This promise object also has methods such as `ready()` and `wait()`,  which allow us to check whether all the results are in yet. I'll leave  you to the Python documentation to discover more about their usage.

Alternatively, if we don't know all the values we want to get results for in advance, we can use the `apply_async`  method to queue up a single job. If the pool has a process that isn't  already working, it will start immediately; otherwise, it will hold onto  the task until there is a free process available.

Pools can also be `closed`, which refuses to take any further tasks, but processes everything currently in the queue, or `terminated`, which goes one step further and refuses to start any jobs still in the queue, although any jobs currently running are still permitted to complete.

# Queues

If we need more control over communication between processes, we can use a `Queue`. `Queue`  data structures are useful for sending messages from one process into  one or more other processes. Any picklable object can be sent into a `Queue`, but remember  that pickling can be a costly operation, so keep such objects small. To  illustrate queues, let's build a little search engine for text content  that stores all relevant entries in memory.

This is not the most  sensible way to build a text-based search engine, but I have used this  pattern to query numerical data that needed to use CPU-intensive  processes to construct a chart that was then rendered to the user.

This  particular search engine scans all files in the current directory in  parallel. A process is constructed for each core on the CPU. Each of  these is instructed to load some of the files into memory. Let's look at  the function that does the loading and searching:

```python
def search(paths, query_q, results_q): 
    lines = [] 
    for path in paths: 
        lines.extend(l.strip() for l in path.open()) 
 
    query = query_q.get() 
    while query: 
        results_q.put([l for l in lines if query in l]) 
        query = query_q.get() 
```

Remember, this function is run in a different process (in fact, it is run in `cpucount()` different processes) from the main thread. It passes a list of `path.path` objects, and two `multiprocessing.Queue` objects; one for incoming queries and one to send outgoing results. These queues automatically  pickle the data in the queue and pass it into the subprocess over a  pipe. These two queues are set up in the main process and passed through  the pipes into the search function inside the child processes.

The  search code is pretty dumb, both in terms of efficiency and of  capabilities; it loops over every line stored in memory and puts the  matching ones in a list. The list is placed in a queue and passed back  to the main process.

Let's look at the `main` process, which sets up these queues:

```python
if __name__ == '__main__': 
    from multiprocessing import Process, Queue, cpu_count 
    from path import path 
    cpus = cpu_count() 
    pathnames = [f for f in path('.').listdir() if f.isfile()] 
    paths = [pathnames[i::cpus] for i in range(cpus)] 
    query_queues = [Queue() for p in range(cpus)] 
    results_queue = Queue() 
     
    search_procs = [ 
        Process(target=search, args=(p, q, results_queue)) 
        for p, q in zip(paths, query_queues) 
    ] 
    for proc in search_procs: proc.start() 
```

For an easier description, let's assume `cpu_count` is four. Notice how the `import` statements are placed inside the `if`  guard? This is a small optimization that prevents them from being  imported in each subprocess (where they aren't needed) on some operating  systems. We list all the paths in the current directory and then split  the list into four approximately equal parts. We also construct a list  of four `Queue` objects to send data into each subprocess. Finally, we construct a **single** results queue. This is passed into all four of the subprocesses. Each of them can put data into the queue and it will be aggregated in the main process.

Now let's look at the code that makes a search actually happen:

```python
    for q in query_queues:
        q.put("def")
        q.put(None) # Signal process termination

    for i in range(cpus):
        for match in results_queue.get():
            print(match)
    for proc in search_procs:
        proc.join()
```

This code performs a single search for `"def"` (because it's a common phrase in a directory full of Python files!).

This  use of queues is actually a local version of what could become a  distributed system. Imagine if the searches were being sent out to  multiple computers and then recombined. Now imagine you had access to  the millions of computers in Google's data centers and you might  understand why they can return search results so quickly!

We won't  discuss it here, but the multiprocessing module includes a manager  class that can take a lot of the boilerplate out of the preceding code.  There is even a version of `multiprocessing.Manager`  that can manage subprocesses on remote systems to construct a  rudimentary distributed application. Check the Python multiprocessing documentation if you are interested in pursuing this further.

# The Problems with Multiprocessing

As threads do, multiprocessing also has problems, some of which we have already discussed. There is no best way to do concurrency; this is especially  true in Python. We always need to examine the parallel problem to  figure out which of the many available solutions is the best one for  that problem. Sometimes, there is no best solution.

In the case of  multiprocessing, the primary drawback is that sharing data between  processes is costly. As we have discussed, all communication between  processes, whether by queues, pipes, or a more implicit mechanism,  requires pickling the objects. Excessive pickling quickly dominates  processing time. Multiprocessing works best when relatively small  objects are passed between processes and a tremendous amount of work  needs to be done on each one. On the other hand, if no communication  between processes is required, there may not be any point in using the  module at all; we can spin up four separate Python processes (by running  each in a separate terminal, for example) and use them independently.

The  other major problem with multiprocessing is that, like threads, it can  be hard to tell which process a variable or method is being accessed in.  In multiprocessing, if you access a variable from another process it  will usually overwrite the variable in the currently running process  while the other process keeps the old value. This is really confusing to  maintain, so don't do it.