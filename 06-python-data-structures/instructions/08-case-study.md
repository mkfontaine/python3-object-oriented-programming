To tie everything together, we'll be writing a simple  link collector, which will visit a website and collect every link on  every page it finds in that site. Before we start, though, we'll need  some test data to work with. Simply write some HTML files to work with  that contain links to each other and to other sites on the internet,  something like this:

```html
<html> 
    <body> 
        <a href="contact.html">Contact us</a> 
        <a href="blog.html">Blog</a> 
        <a href="esme.html">My Dog</a> 
        <a href="/hobbies.html">Some hobbies</a> 
        <a href="/contact.html">Contact AGAIN</a> 
        <a href="http://www.archlinux.org/">Favorite OS</a> 
    </body> 
</html> 
```

Name one of the files `index.html`  so it shows up first when pages are served. Make sure the other files  exist, and keep things complicated so that there is lots of linking  between them. The examples for this chapter include a directory called `case_study_serve` (one of the lamest personal websites in existence!) if you would rather not set them up yourself.

Now, start a simple web server by entering the directory containing all these files and run the following command:

```bash
$python3 -m http.server
```

This will start a server running on port 8000; you can see the pages you made by visiting `http://localhost:8000/` in your web browser.

The goal is to pass our collector the base URL for the site (in this case: `http://localhost:8000/`),  and have it create a list containing every unique link on the site.  We'll need to take into account three types of URLs (links to external  sites, which start with `http://`, absolute internal links, which start with a `/`  character, and relative links, for everything else). We also need to be  aware that pages may link to each other in a loop; we need to be sure  we don't process the same page multiple times, or it may never end. With  all this uniqueness going on, it sounds like we're going to need some  sets.

Before we get into that, let's start with the basics. Here's  the code to connect to a page and parse all the links in that page: 

```python
from urllib.request import urlopen 
from urllib.parse import urlparse 
import re 
import sys 
LINK_REGEX = re.compile( 
        "<a [^>]*href=['\"]([^'\"]+)['\"][^>]*>") 
 
class LinkCollector: 
    def __init__(self, url): 
        self.url = "" + urlparse(url).netloc 
 
    def collect_links(self, path="/"): 
        full_url = self.url + path 
        page = str(urlopen(full_url).read()) 
        links = LINK_REGEX.findall(page) 
        print(links) 
 
if __name__ == "__main__": 
    LinkCollector(sys.argv[1]).collect_links() 
```

This  is a short piece of code, considering what it's doing. It connects to  the server in the argument passed on the command line, downloads the  page, and extracts all the links on that page. The `__init__` method uses the `urlparse` function to extract just the hostname from the URL; so even if we pass in `http://localhost:8000/some/page.html`, it will still operate on the top level of the `http://localhost:8000/`  host. This makes sense, because we want to collect all the links on the  site, although it assumes every page is connected to the index by some  sequence of links.

The `collect_links`  method connects to and downloads the specified page from the server, and  uses a regular expression to find all the links on the page. Regular  expressions are an extremely powerful string processing tool.  Unfortunately, they have a steep learning curve; if you haven't used  them before, I strongly recommend studying any of the many entire books  or websites on the topic. If you don't think they're worth knowing  about, try writing the preceding code without them and you'll change  your mind.

The example also stops in the middle of the `collect_links`  method to print the value of links. This is a common way to test a  program as we're writing it: stop and output the value to ensure it is  the value we expect. Here's what it outputs for our example:

```python
['contact.html', 'blog.html', 'esme.html', '/hobbies.html', 
'/contact.html', 'http://www.archlinux.org/'] 
```

So,  now we have a collection of all the links in the first page. What can we  do with it? We can't just pop the links into a set to remove  duplicates, because links may be relative or absolute. For example, `contact.html` and `/contact.html`  point to the same page. So the first thing we should do is normalize  all the links to their full URL, including the hostname and relative path. We can do this by adding a `normalize_url` method to our object:

```python
    def normalize_url(self, path, link): 
        if link.startswith("http://"): 
            return link 
        elif link.startswith("/"): 
            return self.url + link 
        else: 
            return self.url + path.rpartition( 
                '/')[0] + '/' + link 
```

This method converts each URL to a complete address that includes a protocol and a hostname. Now, the two contact pages have the same value and we can store them in a set. We'll have to modify `__init__` to create the set, and `collect_links` to put all the links into it.

Then,  we'll have to visit all the non-external links and collect them too.  But wait a minute; if we do this, how do we keep from revisiting a link  when we encounter the same page twice? It looks like we're actually  going to need two sets: a set of collected links, and a set of visited  links. This suggests that we were wise to choose a set to represent our  data; we know that sets are most useful when we're manipulating more  than one of them. Let's set these up as follows:

```python
class LinkCollector: 
    def __init__(self, url): 
        self.url = "http://+" + urlparse(url).netloc 
        self.collected_links = set() 
        self.visited_links = set() 
 
    def collect_links(self, path="/"): 
        full_url = self.url + path 
        self.visited_links.add(full_url) 
        page = str(urlopen(full_url).read()) 
        links = LINK_REGEX.findall(page) 
        links = {self.normalize_url(path, link 
            ) for link in links} 
        self.collected_links = links.union( 
                self.collected_links) 
        unvisited_links = links.difference( 
                self.visited_links) 
        print(links, self.visited_links, 
                self.collected_links, unvisited_links) 
```

The line that creates the normalized list of links uses a `set`  comprehension (we'll be covering these in detail in the next chapter).  Once again, the method stops to print out the current values, so we can  verify that we don't have our sets confused, and that `difference` really was the method we wanted to call to collect `unvisited_links`.  We can then add a few lines of code that loop over all the unvisited  links and add them to the collection as well, demonstrated as follows:

```python
        for link in unvisited_links: 
            if link.startswith(self.url): 
                self.collect_links(urlparse(link).path) 
```

The `if`  statement ensures that we are only collecting links from the one  website; we don't want to go off and collect all the links from all the  pages on the internet (unless we're Google or Internet Archive!). If we  modify the main code at the bottom of the program to output the  collected links, we can see it seems to have collected them all, as can  be seen in the following block of code:

```python
if __name__ == "__main__": 
    collector = LinkCollector(sys.argv[1]) 
    collector.collect_links() 
    for link in collector.collected_links: 
        print(link) 
```

It displays all the links we've collected, and only once, even though many of the pages in my example linked to each other multiple times, as follows:

```bash
$ python3 link_collector.py http://localhost:8000http://localhost:8000/http://en.wikipedia.org/wiki/Cavalier_King_Charles_Spanielhttp://beluminousyoga.comhttp://archlinux.me/dusty/http://localhost:8000/blog.htmlhttp://ccphillips.net/http://localhost:8000/contact.htmlhttp://localhost:8000/taichi.htmlhttp://www.archlinux.org/http://localhost:8000/esme.htmlhttp://localhost:8000/hobbies.html
```

Even though it collected links **to** external pages, it didn't go off collecting links **from**  any of the external pages we linked to. This is a great little program  if we want to collect all the links on a site. But it doesn't give me  all the information I might need to build a site map; it tells me which  pages I have, but it doesn't tell me which pages link to other pages. If  we want to do that instead, we're going to have to make some  modifications.

The first thing we should do is look at our data  structures. The set of collected links doesn't work any more; we want to  know which links were linked to from which pages. We can turn that set  into a dictionary of sets for each page we visit. The dictionary keys  will represent the exact same data that is currently in the set. The  values will be sets of all the links on that page. The changes are as follows:

```python
from urllib.request import urlopen 
from urllib.parse import urlparse 
import re 
import sys 
LINK_REGEX = re.compile( 
        "<a [^>]*href=['\"]([^'\"]+)['\"][^>]*>") 
 
class LinkCollector: 
    def __init__(self, url): 
        self.url = "http://%s" % urlparse(url).netloc 
        self.collected_links = {} 
        self.visited_links = set() 
 
    def collect_links(self, path="/"): 
        full_url = self.url + path 
        self.visited_links.add(full_url) 
        page = str(urlopen(full_url).read()) 
        links = LINK_REGEX.findall(page) 
        links = {self.normalize_url(path, link 
            ) for link in links} 
        self.collected_links[full_url] = links 
        for link in links: 
            self.collected_links.setdefault(link, set()) 
        unvisited_links = links.difference( 
                self.visited_links) 
        for link in unvisited_links: 
            if link.startswith(self.url): 
                self.collect_links(urlparse(link).path) 
 
    def normalize_url(self, path, link): 
        if link.startswith("http://"): 
            return link 
        elif link.startswith("/"): 
            return self.url + link 
        else: 
            return self.url + path.rpartition('/' 
                    )[0] + '/' + link 
if __name__ == "__main__": 
    collector = LinkCollector(sys.argv[1]) 
    collector.collect_links() 
    for link, item in collector.collected_links.items(): 
        print("{}: {}".format(link, item)) 
```

There  are surprisingly few changes; the line that originally created a union  of two sets has been replaced with three lines that update the  dictionary. The first of these simply tells the dictionary what the  collected links for that page are. The second creates an empty set for  any items in the dictionary that have not already been added to the dictionary using `setdefault`.  The result is a dictionary that contains all the links as its keys,  mapped to sets of links for all the internal links, and empty sets for  the external links.

Finally, instead of recursively calling `collect_links`,  we can use a queue to store the links that haven't been processed yet.  This implementation won't support concurrency, but this would be a good  first step to creating a multithreaded version that makes multiple  requests in parallel to save time:

```python
from urllib.request import urlopen 
from urllib.parse import urlparse 
import re 
import sys 
from queue import Queue 
LINK_REGEX = re.compile("<a [^>]*href=['\"]([^'\"]+)['\"][^>]*>") 
 
 
class LinkCollector: 
    def __init__(self, url): 
        self.url = "http://%s" % urlparse(url).netloc 
        self.collected_links = {} 
        self.visited_links = set() 
 
    def collect_links(self): 
        queue = Queue() 
        queue.put(self.url) 
        while not queue.empty(): 
            url = queue.get().rstrip('/') 
            self.visited_links.add(url) 
            page = str(urlopen(url).read()) 
            links = LINK_REGEX.findall(page) 
            links = { 
                self.normalize_url(urlparse(url).path, link) 
                for link in links 
            } 
            self.collected_links[url] = links 
            for link in links: 
                self.collected_links.setdefault(link, set()) 
            unvisited_links = links.difference(self.visited_links) 
            for link in unvisited_links: 
                if link.startswith(self.url): 
                    queue.put(link) 
 
    def normalize_url(self, path, link): 
        if link.startswith("http://"): 
            return link.rstrip('/') 
        elif link.startswith("/"): 
            return self.url + link.rstrip('/') 
        else: 
            return self.url + path.rpartition('/')[0] + '/' + link.rstrip('/') 
 
if __name__ == "__main__": 
    collector = LinkCollector(sys.argv[1]) 
    collector.collect_links() 
    for link, item in collector.collected_links.items(): 
        print("%s: %s" % (link, item)) 
```

I had to manually strip any trailing forward slashes in the `normalize_url` method to remove duplicates in this version of the code.

Because  the end result is an unsorted dictionary, there is no restriction on  which order the links should be processed in. Therefore, we could just  as easily have used a `LifoQueue` instead of a `Queue`  here. A priority queue probably wouldn't make a lot of sense, since  there is no obvious priority to attach to a link in this case.