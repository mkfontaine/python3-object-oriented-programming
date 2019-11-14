The strategy pattern is a common demonstration  of abstraction in object-oriented programming. The pattern implements  different solutions to a single problem, each in a different object. The  client code can then choose the most appropriate implementation  dynamically at runtime.

Typically, different algorithms have  different trade-offs; one might be faster than another, but uses a lot  more memory, while a third algorithm may be most suitable when multiple  CPUs are present or a distributed system is provided. Here is the  strategy pattern in UML:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/553da6d1-96c5-41bb-b530-e51e2a1b168a.png)

The **User** code connecting to the strategy pattern simply needs to know that it is dealing with the **Abstraction**  interface. The actual implementation chosen performs the same task, but  in different ways; either way, the interface is identical.

# A Strategy Example

The canonical example of the strategy pattern  is sort routines; over the years, numerous algorithms have been  invented for sorting a collection of objects; quick sort, merge sort,  and heap sort are all fast sort algorithms with different features, each  useful in its own right, depending on the size and type of inputs, how  out of order they are, and the requirements of the system.

If we have client code that needs to sort a collection, we could pass it to an object with a `sort()` method. This object may be a `QuickSorter` or `MergeSorter`  object, but the result will be the same in either case: a sorted list.  The strategy used to do the sorting is abstracted from the calling code,  making it modular and replaceable.

Of course, in Python, we typically just call the `sorted` function or `list.sort` method and trust that it will do the sorting in a near-optimal fashion. So, we really need to look at a better example.

Let's  consider a desktop wallpaper manager. When an image is displayed on a  desktop background, it can be adjusted to the screen size in different  ways. For example, assuming the image is smaller than the screen, it can  be tiled across the screen, centered on it, or scaled to fit. There are  other, more complicated, strategies that can be used as well, such as  scaling to the maximum height or width, combining it with a solid,  semi-transparent, or gradient background color, or other manipulations.  While we may want to add these strategies later, let's start with the  basic ones.

Our strategy objects take two inputs; the image to be  displayed, and a tuple of the width and height of the screen. They each  return a new image the size of the screen, with the image manipulated to  fit according to the given strategy. You'll need to install the `pillow` module with `pip3 install pillow` for this example to work:

```python
from PIL import Image


class TiledStrategy:
    def make_background(self, img_file, desktop_size):
        in_img = Image.open(img_file)
        out_img = Image.new("RGB", desktop_size)
        num_tiles = [
            o // i + 1 for o, i in zip(out_img.size, in_img.size)
        ]
        for x in range(num_tiles[0]):
            for y in range(num_tiles[1]):
                out_img.paste(
                    in_img,
                    (
                        in_img.size[0] * x,
                        in_img.size[1] * y,
                        in_img.size[0] * (x + 1),
                        in_img.size[1] * (y + 1),
                    ),
                )
        return out_img


class CenteredStrategy:
    def make_background(self, img_file, desktop_size):
        in_img = Image.open(img_file)
        out_img = Image.new("RGB", desktop_size)
        left = (out_img.size[0] - in_img.size[0]) // 2
        top = (out_img.size[1] - in_img.size[1]) // 2
        out_img.paste(
            in_img,
            (left, top, left + in_img.size[0], top + in_img.size[1]),
        )
        return out_img


class ScaledStrategy:
    def make_background(self, img_file, desktop_size):
        in_img = Image.open(img_file)
        out_img = in_img.resize(desktop_size)
        return out_img
```

Here we have three strategies, each using `PIL` to perform their task. Individual strategies have a `make_background`  method that accepts the same set of parameters. Once selected, the  appropriate strategy can be called to create a correctly sized version  of the desktop image. `TiledStrategy` loops  over the number of input images that would fit in the width and height  of the image and copies it into each location, repeatedly. `CenteredStrategy` figures out how much space needs to be left on the four edges of the image to center it. `ScaledStrategy` forces the image to the output size (ignoring aspect ratio).

Consider how switching between these options  would be implemented without the strategy pattern. We'd need to put all  the code inside one great big method and use an awkward `if`  statement to select the expected one. Every time we wanted to add a new  strategy, we'd have to make the method even more ungainly.

# Strategy in Python

The preceding canonical implementation of the strategy pattern, while very common in most object-oriented libraries, is rarely seen in Python programming.

These classes each represent objects that do nothing but provide a single function. We could just as easily call that function `__call__`  and make the object callable directly. Since there is no other data  associated with the object, we need do no more than create a set of  top-level functions and pass them around as our strategies instead.

Opponents of design pattern philosophy will therefore say, **because Python has first-class functions, the strategy pattern is unnecessary**.  In truth, Python's first-class functions allow us to implement the  strategy pattern in a more straightforward way. Knowing the pattern  exists can still help us choose a correct design for our program, but  implement it using a more readable syntax. The strategy pattern, or a  top-level function implementation of it, should be used when we need to  allow client code or the end user to select from multiple  implementations of the same interface.