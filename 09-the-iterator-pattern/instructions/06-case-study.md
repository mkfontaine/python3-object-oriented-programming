One of the fields in which Python is the most popular these days is data science. In honor of that fact, let's implement a basic machine learning algorithm.

Machine  learning is a huge topic, but the general idea is to make predictions  or classifications about future data by using knowledge gained from past  data. Uses of such algorithms abound, and data scientists are finding  new ways to apply machine learning every day. Some important machine  learning applications include computer vision (such as image  classification or facial recognition), product recommendation,  identifying spam, and self-driving cars.

So as not to digress into  an entire book on machine learning, we'll look at a simpler problem:  given an RGB color definition, what name would humans identify that  color as?

There are more than 16 million colors in the standard  RGB color space, and humans have come up with names for only a fraction  of them. While there are thousands of names (some quite ridiculous; just  go to any car dealership or paint store), let's build a classifier that  attempts to divide the RGB space into the basic colors:

- Red
- Purple
- Blue
- Green
- Yellow
- Orange
- Gray
- Pink

(In my testing, I classified whitish and blackish colors as gray, and brownish colors as orange.)

The first thing we need is a dataset to train our algorithm on. In a production system, you might scrape a **list of colors**  website or survey thousands of people. Instead, I created a simple  application that renders a random color and asks the user to select one  of the preceding eight options to classify it. I implemented it using `tkinter`,  the user interface toolkit that ships with Python. I'm not going to go  into the details of what this script does, but here it is in its  entirety for completeness (it's a trifle long, so you may want to pull  it from Packt's GitHub repository with the examples for this book  instead of typing it in):

```python
import random
import tkinter as tk
import csv


class Application(tk.Frame):
    def __init__(self, master=None):
        super().__init__(master)
        self.grid(sticky="news")
        master.columnconfigure(0, weight=1)
        master.rowconfigure(0, weight=1)
        self.create_widgets()
        self.file = csv.writer(open("colors.csv", "a"))

    def create_color_button(self, label, column, row):
        button = tk.Button(
            self, command=lambda: self.click_color(label), text=label
        )
        button.grid(column=column, row=row, sticky="news")

    def random_color(self):
        r = random.randint(0, 255)
        g = random.randint(0, 255)
        b = random.randint(0, 255)

        return f"#{r:02x}{g:02x}{b:02x}"

    def create_widgets(self):
        self.color_box = tk.Label(
            self, bg=self.random_color(), width="30", height="15"
        )
        self.color_box.grid(
            column=0, columnspan=2, row=0, sticky="news"
        )
        self.create_color_button("Red", 0, 1)
        self.create_color_button("Purple", 1, 1)
        self.create_color_button("Blue", 0, 2)
        self.create_color_button("Green", 1, 2)
        self.create_color_button("Yellow", 0, 3)
        self.create_color_button("Orange", 1, 3)
        self.create_color_button("Pink", 0, 4)
        self.create_color_button("Grey", 1, 4)
        self.quit = tk.Button(
            self, text="Quit", command=root.destroy, bg="#ffaabb"
        )
        self.quit.grid(column=0, row=5, columnspan=2, sticky="news")

    def click_color(self, label):
        self.file.writerow([label, self.color_box["bg"]])
        self.color_box["bg"] = self.random_color()


root = tk.Tk()
app = Application(master=root)
app.mainloop()
```

info> You  can easily add more buttons for other colors if you like. You may get  tripped up on the layout; the second and third argument to `create_color_button`  represent the row and column of a two column grid that the button goes  in. Once you have all your colors in place, you will want to move the **Quit** button to the last row.

For the purposes of this case study, the important thing to know about this application is the output. It creates a **Comma-Separated Value** (**CSV**) file named `colors.csv`. This file contains two CSVs: the label the user assigned to the color, and the hex RGB value for the color. Here's an example:

```markup
Green,#6edd13
Purple,#814faf
Yellow,#c7c26d
Orange,#61442c
Green,#67f496
Purple,#c757d5
Blue,#106a98
Pink,#d40491
.
.
.
Blue,#a4bdfa
Green,#30882f
Pink,#f47aad
Green,#83ddb2
Grey,#baaec9
Grey,#8aa28d
Blue,#533eda
```

I  made over 250 datapoints before I got bored and decided it was time to  start machine learning on my dataset. My datapoints are shipped with the  examples for this chapter if you would like to use it (nobody's ever  told me I'm colorblind, so it should be somewhat reasonable).

We'll be implementing one of the simpler machine learning algorithms, referred to as **k-nearest neighbor**. This algorithm relies on some kind of **distance** calculation  between points in the dataset (in our case, we can use a  three-dimensional version of the Pythagorean theorem). Given a new  datapoint, it finds a certain number (referred to as **k**, which is the **k** in **k-nearest**)  of datapoints that are closest to it when measured by that distance  calculation. Then it combines those datapoints in some way (an average  might work for linear calculations; for our classification problem,  we'll use the mode), and returns the result.

We won't go into too much detail about what the algorithm does; rather, we'll focus on some of the ways we can apply the iterator pattern or iterator protocol to this problem.

Let's now write a program that performs the following steps in order:

1. Load the sample data from the file and construct a model from it.
2. Generate 100 random colors.
3. Classify each color and output it to a file in the same format as the input.

The first step is a fairly simple generator that loads CSV data and converts it into a format that is amenable to our needs:

```python
import csv

dataset_filename = "colors.csv"


def load_colors(filename):
    with open(filename) as dataset_file:
        lines = csv.reader(dataset_file)
        for line in lines:
            label, hex_color = line
            yield (hex_to_rgb(hex_color), label)
```

We haven't seen the `csv.reader`  function before. It returns an iterator over the lines in the file.  Each value returned by the iterator is a list of strings, as separated by commas. So, the line `Green,#6edd13` is returned as `["Green", "#6edd13"]`.

The `load_colors`  generator then consumes that iterator, one line at a time, and yields a  tuple of RGB values as well as the label. It is quite common for  generators to be chained in this way, where one iterator calls another  that calls another and so on. You may want to look at the `itertools` module in the Python Standard Library for a whole host of such ready-made generators waiting for you.

The  RGB values in this case are tuples of integers between 0 and 255. The  conversion from hex to RGB is a bit tricky, so we pulled it out into a  separate function:

```python
def hex_to_rgb(hex_color):
    return tuple(int(hex_color[i : i + 2], 16) for i in range(1, 6, 2))
```

This generator expression is doing a lot of work. It takes a string such as `"#12abfe"` as input and returns a tuple such as `(18, 171, 254)`. Let's break it down from back to front.

The `range` call will return the numbers `[1, 3, 5]`. These represent the indexes of the three color channels in the hex string. The index, `0`, is skipped, since it represents the character `"#"`, which we don't care about. For each of the three numbers, it extracts the two character string between `i` and `i+2`. For the preceding example string , that would be `12`, `ab`, and `fe`. Then it converts this string value to an integer. The `16` passed as the second argument to the `int` function tells the function to use base-16 (hexadecimal) instead of the usual base-10 (decimal) for the conversion.

Given how difficult the generator expression  is to read, do you think it should have been represented in a different  format? It could be created as a sequence of multiple generator  expressions, for example, or be unrolled into a normal generator  function with `yield` statements. Which would you prefer?

In this case, I am comfortable trusting the function name to explain what the ugly line of code is doing.

Now that we've loaded the **training data** (manually  classified colors, we need some new data to test how well the algorithm  is working. We can do this by generating a hundred random colors, each  composed of three random numbers between 0 and 255.

There are so many ways this can be done:

- A list comprehension with a nested generator expression: `[tuple(randint(0,255) for c in range(3)) for r in range(100)]`
- A basic generator function
- A class that implements the `__iter__` and `__next__` protocols

- Push the data through a pipeline of coroutines
- Even just a basic `for` loop

The generator version seems to be most readable, so let's add that function to our program:

```python
from random import randint
 
def generate_colors(count=100):
    for i in range(count):
        yield (randint(0, 255), randint(0, 255), randint(0, 255))
```

Notice how we parameterize the number of colors to generate. We can now reuse this function for other color-generating tasks in the future.

Now, before we do the classification step, we need a function to calculate the **distance** between two colors. Since it's possible to think of colors as being three dimensional (red, green, and blue could map to the **x**, **y**, and **z** axes, for example), let's use a little basic math:

```python
def color_distance(color1, color2):
    channels = zip(color1, color2)
    sum_distance_squared = 0
    for c1, c2 in channels:
        sum_distance_squared += (c1 - c2) ** 2
    return sum_distance_squared
```

This is a pretty basic-looking function; it doesn't look like it's even using the iterator protocol. There's no `yield` function, no comprehensions. However, there is a `for` loop, and that call to the `zip` function is doing some real iteration as well (if you aren't familiar with it, `zip` yields tuples, each containing one element from each input iterator).

This distance calculation is the three-dimensional version of the Pythagorean theorem you may remember from school: **a<sup>2</sup> + b<sup>2</sup> = c<sup>2</sup>**. Since we are using three dimensions, I guess it would actually be **a<sup>2</sup> + b<sup>2</sup> + c<sup>2</sup> = d<sup>2</sup>**. The distance is technically the square root of **a<sup>2</sup> + b<sup>2</sup> + c<sup>2</sup>**, but there isn't any need to perform the somewhat expensive `sqrt` calculation since the squared distances are all the same relative size to each other.

Now  that we have some plumbing in place, let's do the actual k-nearest  neighbor implementation. This routine can be thought of as consuming and  combining the two generators we've already seen (`load_colors` and `generate_colors`):

```python
def nearest_neighbors(model_colors, target_colors, num_neighbors=5):
    model_colors = list(model_colors)

    for target in target_colors:
        distances = sorted(
            ((color_distance(c[0], target), c) for c in model_colors)
        )
        yield target, distances[:5]
```

We first convert the `model_colors` generator to a list because it has to be consumed multiple times, once for each of the `target_colors`.   If we didn't do this, we would have to load the colors from the source  file repeatedly, which would perform a lot of unnecessary disk reads.

The  downside of this decision is that the entire list has to be stored in  memory all at once. If we had a massive dataset that didn't fit in  memory, it would actually be necessary to reload the generator from disk  each time (though we'd actually be looking at different machine  learning algorithms in that case).

The `nearest_neighbors` generator loops over each target color (a three-tuple, such as `(255, 14, 168)`) and calls the `color_distance` function on it inside a generator expression. The `sorted`  call surrounding that generator expression then sorts the results by  their first element, which is the distance. It is a complicated piece of  code and isn't object-oriented at all. You may want to break it down  into a normal `for` loop to ensure you understand what the generator expression is doing.

The `yield` statement is a bit less complicated. For each RGB three-tuple from the `target_colors` generator, it yields the target, and a list comprehension of the `num_neighbors` (that's the **k** in **k-nearest**,  by the way. Many mathematicians and, by extension, data scientists,  have a horrible tendency to use unintelligible one-letter variable  names) closest colors.

The contents of each element in the list comprehension is an element from the `model_colors`  generator; that is, a tuple of a tuple of three RGB values and the  string name that was manually entered for that color. So, one element  might look like this: `((104, 195, 77), 'Green')`. The first thing I think when I see nested tuples like that is, **that is not the right datastructure**. The RGB color should probably be represented as a named tuple, and the two attributes should maybe go on a dataclass.

We can now add **another** generator to the chain to figure out what name we should give this target color:

```python
from collections import Counter

def name_colors(model_colors, target_colors, num_neighbors=5):
    for target, near in nearest_neighbors(
        model_colors, target_colors, num_neighbors=5
    ):
        print(target, near)
        name_guess = Counter(n[1] for n in near).most_common()[0][0]
        yield target, name_guess
```

This generator is unpacking the tuple returned by `nearest_neighbors` into the three-tuple target and the five nearest datapoints. It uses a `Counter` to find the name that appears most often among the colors that were returned. There is yet another generator expression in the `Counter` constructor; this one extracts the second element (the color name) from each datapoint. Then it yields a tuple RGB value and the guessed name. An example of the return value is `(91, 158, 250) Blue`.

We can write a function that accepts the output from the `name_colors` generator and writes it to a CSV file, with the RGB colors represented as hex values:

```python
def write_results(colors, filename="output.csv"):
    with open(filename, "w") as file:
        writer = csv.writer(file)
        for (r, g, b), name in colors:
            writer.writerow([name, f"#{r:02x}{g:02x}{b:02x}"])
```

This is a function, not a generator. It's consuming the generator in a `for` loop, but it's not yielding anything. It constructs a CSV writer and outputs rows of name, hex value (for example, `Purple,#7f5f95`) pairs for each of the target colors. The only thing that might be confusing in here is the contents of the format string. The `:02x` modifier used with each of the `r`,`g`, and `b` channels outputs the number as a zero-padded two-digit hexadecimal number.

Now  all we have to do is connect these various generators and pipelines  together, and kick off the process with a single function call:

```python
def process_colors(dataset_filename="colors.csv"):
    model_colors = load_colors(dataset_filename)
    colors = name_colors(model_colors, generate_colors(), 5)
    write_results(colors)


if __name__ == "__main__":
    process_colors()
```

So, this function, unlike almost every other function we've defined, is a perfectly normal function without any `yield` statements or `for` loops. It doesn't do any iteration at all.

It does, however, construct three generators. Can you see all three?:

- `load_colors` returns a generator
- `generate_colors` returns a generator
- `name_guess` returns a generator

The `name_guess` generator consumes the first two generators. It, in turn, is then consumed by the `write_results` function.

I  wrote a second Tkinter app to check the accuracy of the algorithm. It  is similar to the first app, except it renders each color and the label  associated with that color. Then you have to manually click **`Yes`** or **`No`**  if the label matches the color. For my example data, I got around 95%  accuracy. This could be improved by implementing the following:

- Adding more color names
- Adding more training data by manually classifying more colors
- Tweaking the value of `num_neighbors`
- Using a more advanced machine learning algorithm

Here's  the code for the output checking app, though I recommend downloading  the example code instead. This would be tedious to type in:

```python
import tkinter as tk
import csv


class Application(tk.Frame):
    def __init__(self, master=None):
        super().__init__(master)
        self.grid(sticky="news")
        master.columnconfigure(0, weight=1)
        master.rowconfigure(0, weight=1)
        self.csv_reader = csv.reader(open("output.csv"))
        self.create_widgets()
        self.total_count = 0
        self.right_count = 0

    def next_color(self):
        return next(self.csv_reader)

    def mk_grid(self, widget, column, row, columnspan=1):
        widget.grid(
            column=column, row=row, columnspan=columnspan, sticky="news"
        )

    def create_widgets(self):
        color_text, color_bg = self.next_color()
        self.color_box = tk.Label(
            self, bg=color_bg, width="30", height="15"
        )
        self.mk_grid(self.color_box, 0, 0, 2)

        self.color_label = tk.Label(self, text=color_text, height="3")
        self.mk_grid(self.color_label, 0, 1, 2)

        self.no_button = tk.Button(
            self, command=self.count_next, text="No"
        )
        self.mk_grid(self.no_button, 0, 2)

        self.yes_button = tk.Button(
            self, command=self.count_yes, text="Yes"
        )
        self.mk_grid(self.yes_button, 1, 2)

        self.percent_accurate = tk.Label(self, height="3", text="0%")
        self.mk_grid(self.percent_accurate, 0, 3, 2)

        self.quit = tk.Button(
            self, text="Quit", command=root.destroy, bg="#ffaabb"
        )
        self.mk_grid(self.quit, 0, 4, 2)

    def count_yes(self):
        self.right_count += 1
        self.count_next()

    def count_next(self):
        self.total_count += 1
        percentage = self.right_count / self.total_count
        self.percent_accurate["text"] = f"{percentage:.0%}"
        try:
            color_text, color_bg = self.next_color()
        except StopIteration:
            color_text = "DONE"
            color_bg = "#ffffff"
            self.color_box["text"] = "DONE"
            self.yes_button["state"] = tk.DISABLED
            self.no_button["state"] = tk.DISABLED
        self.color_label["text"] = color_text
        self.color_box["bg"] = color_bg


root = tk.Tk()
app = Application(master=root)
app.mainloop()
```

You might be wondering, **what does any of this have to do with object-oriented programming? There isn't even one class in this code!**. In some ways, you'd be right; generators are not commonly considered object-oriented. However, the functions that create them return objects; in fact, you could think of those functions as constructors. The constructed object has an appropriate `__next__()`  method. Basically, the generator syntax is a syntax shortcut for a  particular kind of object that would be quite verbose to create without  it.

As a sort of historical note, the second edition of this book  used coroutines to solve this problem instead of basic generators. I  decided to switch it to generators in the third edition for a few  reasons:

- Nobody ever uses coroutines in real life outside of `asyncio`, which we'll cover in the chapter on concurrency. I felt I was wrongly encouraging people to use coroutines to solve problems when they are extremely rarely the right tool.
- The coroutine version is longer and more convoluted with more boilerplate than the generator version.
- The  coroutine version didn't demonstrate enough of the other features  discussed in this chapter, such as list comprehensions and generator  expressions.

In case you might find the  coroutine-based implementation to be historically interesting, I  included a copy of that code with the rest of the downloadable example  code for this lesson.