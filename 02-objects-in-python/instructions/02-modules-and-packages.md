Now we know how to create classes  and instantiate objects. You don't need to write too many classes (or  non-object-oriented code, for that matter) before you start to lose  track of them. For small programs, we can just put all our classes into  one file and add a little script  at the end of the file to start them interacting. However, as our  projects grow, it can become difficult to find the one class that needs  to be edited among the many classes we've defined. This is where **modules**  come in. Modules are simply Python files, nothing more. The single file  in our small program is a module. Two Python files are two modules. If  we have two files in the same folder, we can load a class from one  module for use in the other module.

For example, if we are  building an e-commerce system, we will likely be storing a lot of data  in a database. We can put all the classes and functions related to  database access into a separate file (we'll call it something sensible: `database.py`).  Then, our other modules (for example, customer models, product  information, and inventory) can import classes from that module in order  to access the database.

The `import`  statement is used for importing modules or specific classes or functions  from modules. We've already seen an example of this in our `Point` class in the previous section. We used the `import` statement to get Python's built-in `math` module and use its `sqrt` function in the `distance` calculation.

Here's a concrete example. Assume we have a module called `database.py`, which contains a class called `Database`. A second module called `products.py`  is responsible for product-related queries. At this point, we don't  need to think too much about the contents of these files. What we know  is that `products.py` needs to instantiate the `Database` class from `database.py` so that it can execute queries on the product table in the database.

There are several variations on the `import` statement syntax that can be used to access the class:

```python
import database 
db = database.Database() 
# Do queries on db 
```

This version imports the `database` module into the `products` namespace (the list of names currently accessible in a module or function), so any class or function in the `database` module can be accessed using the `database.<something>` notation. Alternatively, we can import just the one class we need using the `from...import` syntax:

```python
from database import Database 
db = Database() 
# Do queries on db 
```

If, for some reason, `products` already has a class called `Database`, and we don't want the two names to be confused, we can rename the class when used inside the `products` module:

```python
from database import Database as DB 
db = DB() 
# Do queries on db 
```

We can also import multiple items in one statement. If our `database` module also contains a `Query` class, we can import both classes using the following code:

```python
from database import Database, Query
```

Some sources say that we can import all classes and functions from the `database` module using this syntax:

```python
from database import * 
```

**Don't do this.**  Most experienced Python programmers will tell you that you should never  use this syntax (a few will tell you there are some very specific situations where it is useful, but I disagree). They'll use obscure justifications such as **it clutters up the namespace**, which doesn't make much sense to beginners. One way to learn why to avoid this syntax  is to use it and try to understand your code two years later. But we  can save some time and two years of poorly written code with a quick  explanation now!

When we explicitly import the `database` class at the top of our file using `from database import Database`, we can easily see where the `Database` class comes from. We might use `db = Database()` 400 lines later in the file, and we can quickly look at the imports to see where that `Database` class came from. Then, if we need clarification as to how to use the `Database` class, we can visit the original file (or import the module in the interactive interpreter and use the `help(database.Database)` command). However, if we use the `from database import *` syntax, it takes a lot longer to find where that class is located. Code maintenance becomes a nightmare.

In  addition, most code editors are able to provide extra functionality,  such as reliable code completion, the ability to jump to the definition  of a class, or inline documentation, if normal imports are used. The `import *` syntax usually completely destroys their ability to do this reliably.

Finally, using the `import *`  syntax can bring unexpected objects into our local namespace. Sure, it  will import all the classes and functions defined in the module being  imported from, but it will also import any classes or modules that were  themselves imported into that file!

Every name used in a  module should come from a well-specified place, whether it is defined in  that module, or explicitly imported from another module. There should be no magic variables that seem to come out of thin air. We should *always* be able to immediately identify where  the names in our current namespace originated. I promise that if you  use this evil syntax, you will one day have extremely frustrating  moments of **where on earth can this class be coming from?**

info> For fun, try typing `import this`  into your interactive interpreter. It prints a nice poem (with a couple  of inside jokes you can ignore) summarizing some of the idioms that  Pythonistas tend to practice. Specific to this discussion, note the line  **Explicit is better than implicit**. Explicitly importing names into your namespace makes your code much easier to navigate than the implicit `import *` syntax.

# Organizing Modules

As a project grows into a collection  of more and more modules, we may find that we want to add another level  of abstraction, some kind of nested hierarchy on our modules' levels.  However, we can't put modules inside modules; one file can hold only one  file after all, and modules are just files.

Files, however, can go in folders, and so can modules. A **package** is a collection  of modules in a folder. The name of the package is the name of the  folder. We need to tell Python that a folder is a package to distinguish  it from other folders in the directory. To do this, place a (normally  empty) file in the folder named `__init__.py`. If we forget this file, we won't be able to import modules from that folder.

Let's put our modules inside an `ecommerce` package in our working folder, which will also contain a `main.py` file to start the program. Let's additionally add another package inside the `ecommerce` package for various payment options. The folder hierarchy will look like this:

```python
parent_directory/ 
    main.py 
    ecommerce/ 
        __init__.py 
        database.py 
        products.py 
        payments/ 
            __init__.py 
            square.py 
            stripe.py 
```

When importing modules or classes between packages, we have to be cautious about the syntax. In Python 3, there are two ways of importing modules: absolute imports and relative imports.

## Absolute Imports

**Absolute imports** specify the complete path to the module, function, or class we want to import. If we need access to the `Product` class inside the `products` module, we could use any of these syntaxes to perform an absolute import:

```python
import ecommerce.products 
product = ecommerce.products.Product() 

//or

from ecommerce.products import Product 
product = Product() 

//or

from ecommerce import products 
product = products.Product() 
```

The `import` statements use the period operator to separate packages or modules.

These statements will work from any module. We could instantiate a `Product` class using this syntax in `main.py`, in the `database`  module, or in either of the two payment modules. Indeed, assuming the  packages are available to Python, it will be able to import them. For  example, the packages can also be installed in the Python site packages  folder, or the `PYTHONPATH` environment  variable could be customized to dynamically tell Python which folders to  search for packages and modules it is going to import.

So, with  these choices, which syntax do we choose? It depends on your personal  taste and the application at hand. If there are dozens of classes and  functions inside the `products` module that I want to use, I generally import the module name using the `from ecommerce import products` syntax, and then access the individual classes using `products.Product`. If I only need one or two classes from the `products` module, I can import them directly using the `from ecommerce.products import Product` syntax. I don't personally use the first syntax very often, unless I have some kind of name conflict (for example, I need to access two completely different modules called `products` and I need to separate them). Do whatever you think makes your code look more elegant.

## Relative Imports

When working with related modules inside a package, it seems kind of redundant to specify the full path; we know what our parent module is named. This is where **relative imports**  come in. Relative imports are basically a way of saying find a class,  function, or module as it is positioned relative to the current module.  For example, if we are working in the `products` module and we want to import the `Database` class from the `database` module next to it, we could use a relative import:

```python
from .database import Database 
```

The period in front of `database` says **use the database module inside the current package**. In this case, the current package is the package containing the `products.py` file we are currently editing, that is, the `ecommerce` package.

If we were editing the `paypal` module inside the `ecommerce.payments` package, we would want, for example, to **use the database package inside the parent package** instead. This is easily done with two periods, as shown here:

```python
from ..database import Database 
```

We  can use more periods to go further up the hierarchy. Of course, we can  also go down one side and back up the other. We don't have a deep enough  example hierarchy to illustrate this properly, but the following would  be a valid import if we had an `ecommerce.contact` package containing an `email` module and wanted to import the `send_mail` function into our `paypal` module:

```python
from ..contact.email import send_mail 
```

This import uses two periods  indicating, **the parent of the payments package**, and then uses the normal `package.module` syntax to go back down into the contact package.

Finally, we can import code directly from packages, as opposed to just modules inside packages. In this example, we have an `ecommerce` package containing two modules named `database.py` and `products.py`. The database module contains a `db` variable that is accessed from a lot of places. Wouldn't it be convenient if this could be imported as `import ecommerce.db` instead of `import ecommerce.database.db`?

Remember the `__init__.py`  file that defines a directory as a package? This file can contain any  variable or class declarations we like, and they will be available as  part of the package. In our example, if the `ecommerce/__init__.py` file contained the following line:

```python
from .database import db 
```

We could then access the `db` attribute from `main.py` or any other file using the following import:

```python
from ecommerce import db 
```

It might help to think of the `__init__.py` file as if it were an `ecommerce.py`  file, if that file were a module instead of a package. This can also be  useful if you put all your code in a single module and later decide to  break it up into a package of modules. The `__init__.py` file for the new package can still be the main point of contact for other modules talking to it, but the code can be internally organized into several different modules or subpackages.

I recommend not putting much code in an `__init__.py` file, though. Programmers do not expect actual logic to happen in this file, and much like with `from x import *`, it can trip them up if they are looking for the declaration of a particular piece of code and can't find it until they check `__init__.py`.