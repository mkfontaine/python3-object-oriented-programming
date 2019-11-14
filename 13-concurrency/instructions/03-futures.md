Let's start looking at a more asynchronous way of implementing  concurrency. Futures wrap either multiprocessing or threading depending  on what kind of concurrency we need (tending toward I/O versus tending  toward CPU). They don't completely solve the problem of accidentally  altering shared state, but they allow us to structure our code such that  it is easier to track down when we do so.

Futures provide  distinct boundaries between the different threads or processes. Similar  to the multiprocessing pool, they are useful for **call and answer** type  interactions, in which processing can happen in another thread and then  at some point in the future (they are aptly named, after all), you can  ask it for the result. It's really just a wrapper around multiprocessing  pools and thread pools, but it provides a cleaner API and encourages  nicer code.

A future is an object that wraps a function call. That function call is run in the **background**, in a thread or process. The `future` object has methods the main thread can use to check whether the future has completed and to get the results after it has completed.

Let's see another file search example. In the last section, we implemented a version of the `unix grep` command. This time, we'rr create a simple version of the `find` command. The example will search the entire filesystem for paths that contain a given string of characters, as follows:

```python
from concurrent.futures import ThreadPoolExecutor 
from pathlib import Path 
from os.path import sep as pathsep 
from collections import deque 
 
def find_files(path, query_string): 
    subdirs = [] 
    for p in path.iterdir(): 
        full_path = str(p.absolute()) 
        if p.is_dir() and not p.is_symlink(): 
            subdirs.append(p) 
        if query_string in full_path: 
                print(full_path) 
 
    return subdirs 

 
query = '.py' 
futures = deque() 
basedir = Path(pathsep).absolute() 
 
with ThreadPoolExecutor(max_workers=10) as executor: 
    futures.append( 
        executor.submit(find_files, basedir, query)) 
    while futures: 
        future = futures.popleft() 
        if future.exception(): 
            continue 
        elif future.done(): 
            subdirs = future.result() 
            for subdir in subdirs: 
                futures.append(executor.submit( 
                    find_files, subdir, query)) 
        else: 
            futures.append(future) 
```

This code consists of a function named `find_files`, which is run in a separate thread (or process, if we used `ProcessPoolExecutor`  instead). There isn't anything particularly special about this  function, but note how it does not access any global variables. All interaction  with the external environment is passed into the function or returned  from it. This is not a technical requirement, but it is the best way to keep your brain inside your skull when programming with futures.

info> Accessing outside variables without proper synchronization results in something called a **racecondition**.  For example, imagine two concurrent writes trying to increment an  integer counter. They start at the same time and both read the value as  5. Then, they both increment the value and write back the result as 6.  But if two processes are trying to increment a variable, the expected  result would be that it gets incremented by two, so the result should be  7. Modern wisdom is that the easiest way to avoid doing this is to keep  as much state as possible private and share them through known-safe  constructs, such as queues or futures.

We set up a couple of variables before we get started; we'll be searching for all files that contain the characters `'.py'` for this example. We have a queue of futures, which we'll discuss shortly. The `basedir` variable points to the root of the filesystem: `'/'` on Unix machines and probably `C:\` on Windows.

First, let's take a short course on search theory. This algorithm implements breadth-first search in parallel. Rather than recursively searching every directory using a depth-first  search, it adds all the subdirectories in the current folder to the  queue, then all the subdirectories of each of those folders, and so on.

The meat of the program is known as an event loop. We can construct a `ThreadPoolExecutor` as a context manager so that it is automatically cleaned up and closes its threads when it is done. It requires a `max_workers`  argument to indicate the number of threads running at a time. If more  than this many jobs are submitted, it queues up the rest until a worker  thread becomes available. When using `ProcessPoolExecutor`,  this is normally constrained to the number of CPUs on the machine, but  with threads, it can be much higher, depending how many are waiting on  I/O at a time. Each thread takes up a certain amount  of memory, so it shouldn't be too high. It doesn't take all that many  threads before the speed of the disk, rather than the number of parallel  requests, is the bottleneck.

Once the executor has been constructed, we submit a job to it using the root directory. The `submit()` method immediately returns a `Future`  object, which promises to give us a result eventually. The future is  placed in the queue. The loop then repeatedly removes the first future  from the queue and inspects it. If it is still running, it gets added  back to the end of the queue. Otherwise, we check whether the function raised an exception with a call to `future.exception()`.  If it did, we just ignore it (it's usually a permission error, although  a real app would need to be more careful about what the exception was).  If we didn't check this exception here, it would be raised when we  called `result()` and could be handled through the normal `try...except` mechanism.

Assuming no exception occurred, we can call `result()`  to get the return value. Since the function returns a list of  subdirectories that are not symbolic links (my lazy way of preventing an  infinite loop), `result()` returns the same  thing. These new subdirectories are submitted to the executor and the  resulting futures are tossed onto the queue to have their contents  searched in a later iteration.

And that's all that is required to  develop a future-based I/O-bound application. Under the hood, it's using  the same thread or process APIs we've already discussed, but it  provides a more understandable interface and makes it easier to see the  boundaries between concurrently running functions (just don't try to access global variables from inside the future!).