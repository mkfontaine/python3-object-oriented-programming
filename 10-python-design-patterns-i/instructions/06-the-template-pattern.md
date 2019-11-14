The template pattern is useful for removing duplicate code; it's intended to support the **Don't Repeat Yourself** principle we discussed in **When to Use Object-Oriented Programming**. It is designed for situations where we have several different tasks to 
accomplish that have some, but not all, steps in common. The common steps are implemented in a base class, and the distinct steps are overridden in subclasses to provide custom behavior. In some ways, it's like a generalized strategy pattern, except similar sections of the algorithms are shared using a base class. Here it is in the UML format:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/1ee20768-69f3-4b3c-a8a7-6d68273e4989.png)

# A Template Example

Let's create a car sales reporter as an example. We can store records  of sales in an SQLite database table. SQLite is a simple file-based  database engine that allows us to store records using SQL syntax. Python  includes SQLite in its standard library, so there are no extra modules  required.

We have two common tasks we need to perform:

- Select all sales of new vehicles and output them to the screen in a comma-delimited format
- Output  a comma-delimited list of all salespeople with their gross sales and  save it to a file that can be imported to a spreadsheet

These  seem like quite different tasks, but they have some common features. In  both cases, we need to perform the following steps:

1. Connect to the database.
2. Construct a query for new vehicles or gross sales.
3. Issue the query.
4. Format the results into a comma-delimited string.
5. Output the data to a file or email.

The  query construction and output steps are different for the two tasks,  but the remaining steps are identical. We can use the template pattern  to put the common steps in a base class, and the varying steps in two  subclasses.

Before we start, let's create a database and put some sample data in it, using a few lines of SQL:

```python
import sqlite3

conn = sqlite3.connect("sales.db")

conn.execute(
    "CREATE TABLE Sales (salesperson text, "
    "amt currency, year integer, model text, new boolean)"
)
conn.execute(
    "INSERT INTO Sales values"
    " ('Tim', 16000, 2010, 'Honda Fit', 'true')"
)
conn.execute(
    "INSERT INTO Sales values"
    " ('Tim', 9000, 2006, 'Ford Focus', 'false')"
)
conn.execute(
    "INSERT INTO Sales values"
    " ('Gayle', 8000, 2004, 'Dodge Neon', 'false')"
)
conn.execute(
    "INSERT INTO Sales values"
    " ('Gayle', 28000, 2009, 'Ford Mustang', 'true')"
)
conn.execute(
    "INSERT INTO Sales values"
    " ('Gayle', 50000, 2010, 'Lincoln Navigator', 'true')"
)
conn.execute(
    "INSERT INTO Sales values"
    " ('Don', 20000, 2008, 'Toyota Prius', 'false')"
)
conn.commit()
conn.close()
```

Hopefully, you can see what's going on here even if you don't know SQL; we've created a table to hold the data, and used six `insert` statements to add sales records. The data is stored in a file named `sales.db`. Now we have a sample we can work with in developing our template pattern.

Since  we've already outlined the steps that the template has to perform, we  can start by defining the base class that contains the steps. Each step  gets its own method (to make it easy to selectively override any one  step), and we have one more managerial method that calls the steps in  turn. Without any method content, here's how it might look:

```python
class QueryTemplate:
    def connect(self):
        pass

    def construct_query(self):
        pass

    def do_query(self):
        pass

    def format_results(self):
        pass

    def output_results(self):
        pass

    def process_format(self):
        self.connect()
        self.construct_query()
        self.do_query()
        self.format_results()
        self.output_results()
```

The `process_format`  method is the primary method to be called by an outside client. It  ensures each step is executed in order, but it does not care whether  that step is implemented in this class or in a subclass. For our  examples, we know that three methods are going to be identical between  our two classes:

```python
import sqlite3 
 
class QueryTemplate:
    def connect(self):
        self.conn = sqlite3.connect("sales.db")

    def construct_query(self):
        raise NotImplementedError()

    def do_query(self):
        results = self.conn.execute(self.query)
        self.results = results.fetchall()

    def format_results(self):
        output = []
        for row in self.results:
            row = [str(i) for i in row]
            output.append(", ".join(row))
        self.formatted_results = "\n".join(output)

    def output_results(self):
        raise NotImplementedError()
```

To help with implementing subclasses, the two methods that are not specified raise `NotImplementedError`.  This is a common way to specify abstract interfaces in Python when  abstract base classes seem too heavyweight. The methods could have empty  implementations (with `pass`), or could be fully unspecified. Raising `NotImplementedError`,  however, helps the programmer understand that the class is meant to be  subclassed and these methods overridden. Empty methods or methods that  do not exist are harder to identify as needing to be implemented and to  debug if we forget to implement them.

Now we have a template class  that takes care of the boring details, but is flexible enough to allow  the execution and formatting of a wide variety of queries. The best part  is, if we ever want to change our database engine from SQLite to  another database engine (such as `py-postgresql`), we only have to do it here, in this template class, and we don't have to touch the two (or two hundred) subclasses we might have written.

Let's have a look at the concrete classes now:

```python
import datetime 

class NewVehiclesQuery(QueryTemplate):
    def construct_query(self):
        self.query = "select * from Sales where new='true'"

    def output_results(self):
        print(self.formatted_results)


class UserGrossQuery(QueryTemplate):
    def construct_query(self):
        self.query = (
            "select salesperson, sum(amt) "
            + " from Sales group by salesperson"
        )

    def output_results(self):
        filename = "gross_sales_{0}".format(
            datetime.date.today().strftime("%Y%m%d")
        )
        with open(filename, "w") as outfile:
            outfile.write(self.formatted_results)
```

These  two classes are actually pretty short, considering what they're doing:  connecting to a database, executing a query, formatting the results, and  outputting them. The superclass takes care of the repetitive work, but  lets us easily specify those steps that vary between tasks. Further, we  can also easily change steps that are provided in the base class. For  example, if we wanted to output something other than a comma-delimited  string (for example: an HTML report to be uploaded to a website), we can  still override `format_results`.