Python ships with a lovely standard library, which is a collection  of packages and modules that are available on every machine that runs  Python. However, you'll soon find that it doesn't contain everything you  need. When this happens, you have two options:

- Write a supporting package yourself
- Use somebody else's code

We  won't be covering the details about turning your packages into  libraries, but if you have a problem you need to solve and you don't  feel like coding it (the best programmers are extremely lazy and prefer  to reuse existing, proven code, rather than write their own), you can  probably find the library you want on the **Python Package Index** (**PyPI**) [here](<http://pypi.python.org/>). Once you've identified a package that you want to install, you can use a tool called `pip` to install it. However, `pip` does not come with Python, but Python 3.4 and higher contain a useful tool called `ensurepip`. You can use this command to install it:

```bash
$python -m ensurepip
```

This  may fail for you on Linux, macOS, or other Unix systems, in which case,  you'll need to become a root user to make it work. On most modern Unix  systems, this can be done with `sudo python -m ensurepip`.

info> If you are using an older version of Python than Python 3.4, you'll need to download and install `pip` yourself, since `ensurepip` isn't available. You can do this by following the instructions [here](<http://pip.readthedocs.org/>).

Once `pip` is installed and you know the name of the package you want to install, you can install it using syntax such as the following:

```bash
$pip install requests
```

However,  if you do this, you'll either be installing the third-party library  directly into your system Python directory, or, more likely, will get an  error that you don't have permission to do so. You could force the  installation as an administrator, but common consensus in the Python  community is that you should only use system installers to install the  third-party library to your system Python directory.

Instead, Python 3.4 (and higher) supplies the `venv` tool. This utility basically gives you a mini Python installation called a **virtual environment**  in your working directory. When you activate the mini Python, commands  related to Python will work on that directory instead of the system  directory. So, when you run `pip` or `python`, it won't touch the system Python at all. Here's how to use it:

```bash
cd project_directorypython -m venv envsource env/bin/activate  # on Linux or macOSenv/bin/activate.bat     # on Windows
```

Typically,  you'll create a different virtual environment for each Python project  you work on. You can store your virtual environments anywhere, but I  traditionally keep mine in the same directory as the rest of my project  files (but ignored in version control), so first we `cd` into that directory. Then, we run the `venv` utility to create a virtual environment named `env`.  Finally, we use one of the last two lines (depending on the operating  system, as indicated in the comments) to activate the environment. We'll  need to execute this line each time we want to use that particular  virtualenv, and then use the `deactivate`command when we are done working on this project.

Virtual  environments are a terrific way to keep your third-party dependencies  separate. It is common to have different projects that depend on  different versions of a particular library (for example, an older  website might run on Django 1.8, while newer versions run on Django  2.1). Keeping each project in separate virtualenvs makes it easy to work  in either version of Django. Furthermore, it prevents conflicts between  system-installed packages and `pip`-installed packages if you try to install the same package using different tools.

info> There are several third-party tools for managing virtual environments effectively. Some of these include `pyenv`, `virtualenvwrapper`, and `conda`. My personal preference at the time of writing is `pyenv`, but there is no clear winner here. Do a quick web search and see what works for you.