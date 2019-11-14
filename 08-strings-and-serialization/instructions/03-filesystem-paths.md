All operating systems provide a **filesystem**, a way of mapping a logical abstraction of **folders** (or **directories**) and **files** to the bits and bytes stored  on a hard drive or other storage device. As humans, we typically  interact with the filesystem using a drag-and-drop interface of folders  and files of different types, or with command-line programs such as `cp`, `mv`, and `mkdir`.

As  programmers, we have to interact with the filesystem with a series of  system calls. You can think of these as library functions supplied by  the operating system so that programs can call them. They have a clunky  interface with integer file handles and buffered reads and writes, and  that interface is different depending on which operating system you are  using. Python provides an OS-independent abstraction over these system  calls in the `os.path` module. It's a little  easier to work with than accessing the operating system directly, but  it's not very intuitive. It requires a lot of string concatenation and  you have to be conscious of whether to use a forward slash or a  backslash between directories, depending on the operating system. There  is a `os.sep` file representing the path separator, but using it requires code that looks like this:

```python
>>> path = os.path.abspath(os.sep.join(['.', 'subdir', 'subsubdir', 'file.ext']))
>>> print(path)
/home/dusty/subdir/subsubdir/file.ext
```

Working  with filesystem paths is easily one of the most irritating uses of  strings inside the entire standard library. Paths that are easy to type  on the command  line become illegible in Python code. When you have to manipulate and  access multiple paths (for example, when processing images in a data  pipeline for a machine learning computer vision problem), just managing  those directories becomes a bit of an ordeal.

So, the Python language designers included a module called `pathlib`  in the standard library. It's an object-oriented representation of  paths and files that is much more pleasant to work with. The preceding  path, using `pathlib`, would look like this:

```python
>>> path = (pathlib.Path(".") / "subdir" / " subsubdir" / "file.ext").absolute()
>>> print(path)
/home/dusty/subdir/subsubdir/file.ext
```

As  you can see, it's quite a bit easier to see what's going on. Notice the  unique use of the division operator as a path separator so you don't  have to do anything with `os.sep`.

In a  more real-world example, consider some code that counts the number of  lines of code excluding whitespace and comments in all Python files in  the current directory and subdirectories:

```python
import pathlib


def count_sloc(dir_path):
    sloc = 0
    for path in dir_path.iterdir():
        if path.name.startswith("."):
            continue
        if path.is_dir():
            sloc += count_sloc(path)
            continue
        if path.suffix != ".py":
            continue
        with path.open() as file:
            for line in file:
                line = line.strip()
                if line and not line.startswith("#"):
                    sloc += 1
    return sloc


root_path = pathlib.Path(".")

print(f"{count_sloc(root_path)} lines of python code")
```

In typical `pathlib` usage, we rarely have to construct  more than one or two paths. Usually, other files or directories are  relative to a general path. This example demonstrates that. We only ever  construct one path, from the current directory using `pathlib.Path(".")`. Then, other paths are created based on that path.

The `count_sloc` function first initializes the **sloc** (**source lines of code**) counter to zero. Then, it iterates over all the files and directories in the path that was passed into the function using the `dir_path.iterdir`  generator (we'll discuss generators in detail in the next chapter; for  now, think of it as a sort of dynamic list). Each of the paths returned  to the `for` loop by `iterdir` is itself another path. We first test whether this path starts with a `.`, which represents a hidden directory on most OSes (this will keep it from counting any files in the `.git` directory if you are using version control). Then, we check whether it is a directory using the `isdir()`  method. If it is, we recursively call `count_sloc` to count the lines of code in modules in the child package.

If it's not a directory, we assume it is a normal file, and skip any files that don't end with the `.py` extension, using the `suffix` property. Now, knowing we have a path to a Python file, we open the file using the `open()` method, which returns a context manager. We wrap this in a `with` block so the file is automatically closed when we are done with it.

The `Path.open` method takes similar arguments to the `open` built-in function, but it uses a more object-oriented syntax. If you prefer the function version, you can pass a `Path` object into it as the first parameter (in other words, `with open(Path('./README.md')):`) just as you would a string. But I personally think `Path('./README.md').open()` is more legible if the path already exists.

We  then iterate over each line in the file and add it to the count. We  skip whitespace and comment lines, since these don't represent actual  source code. The total count is returned to the calling function, which may be the original call or the recursive parent.

The `Path` class in the `pathlib`  module has a method or property to cover pretty much everything you  might want to do with a path. In addition to those we covered in the  example, here are a few of my favorites:

- `.absolute()` returns the full path  from the root of the filesystem. I usually call this on every path I  construct in due to a bit of paranoia that I might forget where relative  paths came from.
- `.parent` returns a path to the parent directory.
- `.exists()` checks whether the file or directory exists.
- `.mkdir()` creates a directory at the current path. It takes Boolean `parents` and `exist_ok`  arguments to indicate that it should recursively create the directories  if necessary and that it shouldn't raise an exception if the directory  already exists.

Refer to the standard library documentation [here](<https://docs.python.org/3/library/pathlib.html>) for more exotic uses.

Most standard library modules that accept a string path can also accept a `pathlib.Path` object. For example, you can open a ZIP file by passing a path into it:

```python
>>> zipfile.ZipFile(Path('nothing.zip'), 'w').writestr('filename', 'contents')
```

This  doesn't always work, especially if you are using a third-party library  that is implemented as a C extension. In those cases, you'll have to  cast the path to a string using `str(pathname)`.