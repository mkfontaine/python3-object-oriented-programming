We've been focused on objects and  their attributes and methods. Now, we'll take a look at designing  higher-level objects; the kind of objects that manage other objects –  the objects that tie everything together.

The difference between  these objects and most of the previous examples is that the latter  usually represent concrete ideas. Management objects are more like  office managers; they don't do the actual **visible** work  out on the floor, but without them, there would be no communication  between departments and nobody would know what they are supposed to do  (although, this can be true anyway if the organization is badly  managed!). Analogously, the attributes on a management class tend to  refer to other objects that do the **visible** work; the behaviors on such a class delegate to those other classes at the right time, and pass messages between them.

As  an example, we'll write a program that does a find-and-replace action  for text files stored in a compressed ZIP file. We'll need objects to  represent the ZIP file and each individual text file (luckily, we don't  have to write these classes, as they're available in the Python standard  library). The manager object will be responsible for ensuring the  following three steps occur in order:

1. Unzipping the compressed file
2. Performing the find-and-replace action
3. Zipping up the new files

The class is initialized with the `.zip`  filename, and search and replace strings. We create a temporary  directory to store the unzipped files in, so that the folder stays  clean. The `pathlib` library helps out with file and directory manipulation. We'll learn more about it in **Strings and Serialization**, but the interface should be pretty clear in the following example:

```python
import sys 
import shutil 
import zipfile 
from pathlib import Path 
 
class ZipReplace: 
    def __init__(self, filename, search_string, replace_string): 
        self.filename = filename 
        self.search_string = search_string 
        self.replace_string = replace_string 
        self.temp_directory = Path(f"unzipped-{filename}")
```

Then, we create an overall **manager** method for each of the three steps. This method delegates responsibility to other objects:

```python
def zip_find_replace(self): 
    self.unzip_files() 
    self.find_replace() 
    self.zip_files() 
```

Obviously, we could do all  three steps in one method, or indeed in one script, without ever  creating an object. There are several advantages to separating the three steps:

- **Readability**:  The code for each step is in a self-contained unit that is easy to read  and understand. The method name describes what the method does, and  less additional documentation is required to understand what is going  on.
- **Extensibility**: If a subclass wanted to use compressed TAR files instead of ZIP files, it could override the `zip` and `unzip` methods without having to duplicate the `find_replace` method.
- **Partitioning**: An external class could create an instance of this class and call the `find_replace` method directly on some folder without having to `zip` the content.

The delegation method is the first in the following code; the rest of the methods are included for completeness:

```python
    def unzip_files(self):
        self.temp_directory.mkdir()
        with zipfile.ZipFile(self.filename) as zip:
            zip.extractall(self.temp_directory)

    def find_replace(self):
        for filename in self.temp_directory.iterdir():
            with filename.open() as file:
                contents = file.read()
            contents = contents.replace(self.search_string, self.replace_string)
            with filename.open("w") as file:
                file.write(contents)

    def zip_files(self):
        with zipfile.ZipFile(self.filename, "w") as file:
            for filename in self.temp_directory.iterdir():
                file.write(filename, filename.name)
        shutil.rmtree(self.temp_directory)


if __name__ == "__main__":
    ZipReplace(*sys.argv[1:4]).zip_find_replace()
```

For  brevity, the code for zipping and unzipping files is sparsely  documented. Our current focus is on object-oriented design; if you are  interested in the inner details of the `zipfile` module, refer to the documentation in the standard library, either online or by typing `import zipfile ; help(zipfile)`  into your interactive interpreter. Note that this toy example only  searches the top-level files in a ZIP file; if there are any folders in  the unzipped content, they will not be scanned, nor will any files  inside those folders.

info> If you are using a Python version older than 3.6, you will need to convert the path objects to strings before calling `extractall`, `rmtree`, and `file.write` on the `ZipFile` object.

The last two lines in the example allow us to run the program from the command line by passing the `zip` filename, the search string, and the replace string as arguments, as follows:

```bash
$python zipsearch.py hello.zip hello hi
```

Of course, this object does not have to be created  from the command line; it could be imported from another module (to  perform batch ZIP file processing), or accessed as part of a GUI  interface or even a higher-level management object that knows where to  get ZIP files (for example, to retrieve them from an FTP server or back  them up to an external disk).

As programs become more and more  complex, the objects being modeled become less and less like physical  objects. Properties are other abstract objects, and methods are actions  that change the state of those abstract objects. But at the heart of  every object, no matter how complex, is a set of concrete data and  well-defined behaviors.

# Removing Duplicate Code

Often, the code in management style classes such as `ZipReplace`  is quite generic and can be applied in a variety of ways. It is  possible to use either composition or inheritance to help keep this code  in one place, thus eliminating duplicate code. Before we look at any  examples of this, let's discuss a tiny bit of theory. Specifically, why  is duplicate code a bad thing?

There are several reasons, but they  all boil down to readability and maintainability. When we're writing a  new piece of code that is similar to an earlier piece, the easiest thing  to do is copy the old code and change whatever needs to be changed  (variable names, logic, comments) to make it work in the new location.  Alternatively, if we're writing new code that seems similar, but not  identical, to code elsewhere in the project, it is often easier to write  fresh code with similar behavior, rather than figuring out how to  extract the overlapping functionality.

But as soon as someone has  to read and understand the code and they come across duplicate blocks,  they are faced with a dilemma. Code that might have appeared to make  sense suddenly has to be understood. How is one section different from  the other? How are they the same? Under what conditions is one section  called? When do we call the other? You might argue that you're the only  one reading your code, but if you don't touch that code for eight  months, it will be as incomprehensible to you as it is to a fresh coder.  When we're trying to read two similar pieces of code, we have to  understand why they're different, as well as how they're different. This  wastes the reader's time; code should always be written to be readable  first.

info> I  once had to try to understand someone's code that had three identical  copies of the same 300 lines of very poorly written code. I had been  working with the code for a month before I finally comprehended that the  three **identical** versions were  actually performing slightly different tax calculations. Some of the  subtle differences were intentional, but there were also obvious areas  where someone had updated a calculation in one function without updating  the other two. The number of subtle, incomprehensible bugs in the code  could not be counted. I eventually replaced all 900 lines with an  easy-to-read function of 20 lines or so.

Reading such duplicate code can be tiresome, but code maintenance  is even more tormenting. As the preceding story suggests, keeping two  similar pieces of code up to date can be a nightmare. We have to  remember to update both sections whenever we update one of them, and we  have to remember how multiple sections differ so we can modify our  changes when we are editing each of them. If we forget to update all  sections, we will end up with extremely annoying bugs that usually  manifest themselves as, But I fixed that already, why is it still  happening*?*

The result is  that people who are reading or maintaining our code have to spend  astronomical amounts of time understanding and testing it compared to  the time required to write it in a non-repetitive manner in the first  place. It's even more frustrating when we are the ones doing the  maintenance; we find ourselves saying, Why didn't I do this right the  first time? The time we save by copying and pasting existing code is  lost the very first time we have to maintain it. Code is both read and  modified many more times and much more often than it is written.  Comprehensible code should always be a priority.

This is why  programmers, especially Python programmers (who tend to value elegant  code more than average developers), follow what is known as the **Don't Repeat Yourself** (**DRY**)  principle. DRY code is maintainable code. My advice for beginning  programmers is to never use the copy-and-paste feature of their editor.  To intermediate programmers, I suggest they think thrice before they hit  ***Ctrl*** + ***C***.

But  what should we do instead of code duplication? The simplest solution is  often to move the code into a function that accepts parameters to  account for whatever parts are different. This isn't a terribly  object-oriented solution, but it is frequently optimal.

For  example, if we have two pieces of code that unzip a ZIP file into two  different directories, we can easily replace it with a function that  accepts a parameter  for the directory to which it should be unzipped. This may make the  function itself slightly more difficult to read, but a good function  name and docstring can easily make up for that, and any code that  invokes the function will be easier to read.

That's certainly  enough theory! The moral of the story is: always make the effort to  refactor your code to be easier to read instead of writing bad code that  may seem easier to write.

# In Practice

Let's  explore two ways we can reuse existing code. After writing our code to  replace strings in a ZIP file full of text files, we are later contracted to scale all the images in a ZIP file to 640 x 480. It looks like we could use a very similar paradigm to what we used in `ZipReplace`. Our first impulse might be to save a copy of that file and change the `find_replace` method to `scale_image` or something similar.

But, that's suboptimal. What if someday we want to change the `unzip` and `zip`  methods to also open TAR files? Or maybe we'll want to use a guaranteed  unique directory name for temporary files. In either case, we'd have to  change it in two different places!

We'll start by demonstrating an inheritance-based solution to this problem. First, we'll modify our original `ZipReplace` class into a superclass for processing generic ZIP files:

```python
import sys
import shutil
import zipfile
from pathlib import Path


class ZipProcessor:
    def __init__(self, zipname):
        self.zipname = zipname
        self.temp_directory = Path(f"unzipped-{zipname[:-4]}")

    def process_zip(self):
        self.unzip_files()
        self.process_files()
        self.zip_files()

    def unzip_files(self):
        self.temp_directory.mkdir()
        with zipfile.ZipFile(self.zipname) as zip:
            zip.extractall(self.temp_directory)

    def zip_files(self):
        with zipfile.ZipFile(self.zipname, "w") as file:
            for filename in self.temp_directory.iterdir():
                file.write(filename, filename.name)
        shutil.rmtree(self.temp_directory)
```

We changed the `filename` property to `zipname` to avoid confusion with the `filename` local variables inside the various methods. This helps make the code more readable, even though it isn't actually a change in design.

We also dropped the two parameters to `__init__` (`search_string` and `replace_string`) that were specific to `ZipReplace`. Then, we renamed the `zip_find_replace` method to `process_zip` and made it call an (as yet undefined) `process_files` method instead of `find_replace`; these name changes help demonstrate the more generalized nature of our new class. Notice that we have removed the `find_replace` method altogether; that code is specific to `ZipReplace` and has no business here.

This new `ZipProcessor` class doesn't actually define a `process_files`  method. If we ran it directly, it would raise an exception. Because it  isn't meant to run directly, we removed the main call at the bottom of  the original script. We could make this an abstract base class in order  to communicate that this method needs to be defined in a subclass, but  I've left it out for brevity.

Now, before we move on to our image processing application, let's fix up our original `zipsearch` class to make use of this parent class, as follows:

```python
class ZipReplace(ZipProcessor):
    def __init__(self, filename, search_string, replace_string):
        super().__init__(filename)
        self.search_string = search_string
        self.replace_string = replace_string

    def process_files(self):
        """perform a search and replace on all files in the
        temporary directory"""
        for filename in self.temp_directory.iterdir():
            with filename.open() as file:
                contents = file.read()
            contents = contents.replace(self.search_string, self.replace_string)
            with filename.open("w") as file:
                file.write(contents)
```

This code is  shorter than the original version, since it inherits its ZIP processing  abilities from the parent class. We first import the base class we just wrote and make `ZipReplace` extend that class. Then, we use `super()` to initialize the parent class. The `find_replace` method is still here, but we renamed it `process_files`  so the parent class can call it from its management interface. Because  this name isn't as descriptive as the old one, we added a docstring to  describe what it is doing.

Now, that was quite a bit of work,  considering that all we have now is a program that is functionally not  different from the one we started with! But having done that work, it is  now much easier for us to write other classes that operate on files in a  ZIP archive, such as the (hypothetically requested) photo scaler.  Further, if we ever want to improve or bug fix the zip functionality, we  can do it for all subclasses at once by changing only the one `ZipProcessor` base class. Therefore maintenance will be much more effective.

See how simple it is now to create a photo scaling class that takes advantage of the `ZipProcessor` functionality:

```python
from PIL import Image 
 
class ScaleZip(ZipProcessor): 
  
    def process_files(self): 
        '''Scale each image in the directory to 640x480''' 
        for filename in self.temp_directory.iterdir(): 
            im = Image.open(str(filename)) 
            scaled = im.resize((640, 480)) 
            scaled.save(filename)
 
if __name__ == "__main__": 
    ScaleZip(*sys.argv[1:4]).process_zip() 
```

Look  how simple this class is! All that work we did earlier paid off. All we  do is open each file (assuming that it is an image; it will unceremoniously crash if a file cannot be opened or isn't an image), scale it, and save it back. The `ZipProcessor` class takes care of the zipping and unzipping without any extra work on our part.