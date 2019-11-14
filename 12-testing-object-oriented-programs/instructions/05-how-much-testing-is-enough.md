We've already established that untested  code is broken code. But how can we tell how well our code is tested?  How do we know how much of our code is actually being tested and how  much is broken? The first question is the more important one, but it's  hard to answer. Even if we know we have tested every line of code in our  application, we do not know that we have tested it properly. For  example, if we write a stats test that only checks what happens when we  provide a list of integers, it may still fail spectacularly if used on a  list of floats, strings, or self-made objects. The onus of designing  complete test suites still lies with the programmer.

The second question–how much of our code is actually being tested–is easy to verify. **Code coverage** is an estimate of the number of lines of code that are executed by a program. If we know  that number and the number of lines that are in the program, we can get  an estimate of what percentage of the code was really tested, or  covered. If we additionally have an indicator as to which lines were not  tested, we can more easily write new tests to ensure those lines are  less broken.

The most popular tool for testing code coverage is called, memorably enough, `coverage.py`. It can be installed like most other third-party libraries, using the `pip install coverage `command.

We  don't have space to cover all the details of the coverage API, so we'll  just look at a few typical examples. If we have a Python script that  runs all our unit tests for us (for example, using `unittest.main`,  `discover`, `pytest`, or a custom test runner), we can use the following command to perform a coverage analysis:

```bash
$coverage run coverage_unittest.py
```

This command will exit normally, but it creates a file named `.coverage`, which holds the data from the run. We can now use the `coverage``report` command to get an analysis of the code coverage:

```bash
$coverage report
```

The resulting output should be as follows:

```markup
Name                           Stmts   Exec  Cover--------------------------------------------------coverage_unittest                  7      7   100%stats                             19      6    31%--------------------------------------------------TOTAL                             26     13    50%
```

This  basic report lists the files that were executed (our unit test and a  module it imported). The number of lines of code in each file, and the  number that were executed by the test are also listed. The two numbers  are then combined to estimate the amount of code coverage. If we pass  the `-m` option to the `report` command, it will additionally add a column that looks as follows:

```markdown
Missing-----------8-12, 15-23
```

The ranges of lines listed here identify lines in the `stats` module that were not executed during the test run.

The  example we just ran the code coverage tool on uses the same stats  module we created earlier in the lesson. However, it deliberately uses a  single test that fails to test a lot of code in the file. Here's the  test:

```python
from stats import StatsList 
import unittest 
 
class TestMean(unittest.TestCase): 
    def test_mean(self): 
        self.assertEqual(StatsList([1,2,2,3,3,4]).mean(), 2.5) 
 
if __name__ == "__main__": 
 
    unittest.main() 
```

This code doesn't test the median or mode functions, which correspond to the line numbers that the coverage output told us were missing.

The textual report provides sufficient information, but if we use the `coverage html` command,  we can get an even more useful interactive HTML report, which we can  view in a web browser. The web page even highlights which lines in the  source code were and were not tested. Here's how it looks:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/37932e64-0f4f-46bd-9abc-e2cb687ff69e.png)

We can use the `coverage.py` module with `pytest` as well. We'll need to install the `pytest` plugin for code coverage, using `pip install pytest-coverage`. The plugin adds several command-line options to `pytest`, the most useful being `--cover-report`, which can be set to `html`, `report`, or `annotate` (the latter actually modifies the original source code to highlight any lines that were not covered).

Unfortunately,  if we could somehow run a coverage report on this section of the  chapter, we'd find that we have not covered most of what there is to  know about code coverage! It is possible to use the coverage API to  manage code coverage from within our own programs (or test suites), and `coverage.py` accepts numerous configuration options that we haven't touched on. We also haven't discussed the difference between statement coverage and branch coverage (the latter is much more useful, and the default in recent versions of `coverage.py`), or other styles of code coverage.

Bear  in mind that while 100 percent code coverage is a lofty goal that we  should all strive for, 100 percent coverage is not enough! Just because a  statement was tested does not mean that it was tested properly for all  possible inputs.