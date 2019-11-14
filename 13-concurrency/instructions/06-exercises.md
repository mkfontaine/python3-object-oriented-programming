We've covered several different concurrency paradigms in this lesson and still don't have a clear idea of when each one is useful. As we saw  in the case study, it is often a good idea to prototype a few different  strategies before committing to one.

Concurrency in Python 3 is a  huge topic and an entire course of this size could not cover everything  there is to know about it. As your first exercise, I encourage you to  search the web to discover what are considered to be the latest Python  concurrency best practices.

If you have used threads in a recent application, take a look at the code and see how you can make it more readable and less bug-prone by using futures. Compare thread and multiprocessing futures to see whether you can gain anything by using multiple CPUs.

Try  implementing an AsyncIO service for some basic HTTP requests. If you  can get it to the point that a web browser can render a simple GET  request, you'll have a good understanding of AsyncIO network transports  and protocols.

Make sure you understand the race conditions that  happen in threads when you access shared data. Try to come up with a  program that uses multiple threads to set shared values in such a way  that the data deliberately becomes corrupt or invalid.

Remember the link collector we covered for the case study in **Python Data Structures**? Can you make it run faster by making requests in parallel? Is it better to use raw threads, futures, or AsyncIO for this?

Try  writing the run-length encoding example using threads or  multiprocessing directly. Do you get any speed gains? Is the code easier  or harder to reason about? Is there any way to speed up the  decompression script by using concurrency or parallelism?