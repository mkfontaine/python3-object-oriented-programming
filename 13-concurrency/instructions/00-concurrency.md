Concurrencyis the art of making a computer do (or appear to do) multiple things at once. Historically, this meant inviting the processor to switch between  different tasks many times per second. In modern systems, it can also  literally mean doing two or more things simultaneously on separate  processor cores.

Concurrency is not inherently an object-oriented  topic, but Python's concurrent systems provide object-oriented  interfaces, as we've covered throughout the book. This chapter will  introduce you to the following topics:

- Threads
- Multiprocessing
- Futures
- AsyncIO

Concurrency  is complicated. The basic concepts are fairly simple, but the bugs that  can occur are notoriously difficult to track down. However, for many  projects, concurrency is the only way to get the performance we need.  Imagine if a web server couldn't respond to a user's request until  another user had completed! We won't be going into all the details of  just how hard it is (another full book would be required), but we'll see how to implement basic concurrency in Python, and some common pitfalls to avoid.