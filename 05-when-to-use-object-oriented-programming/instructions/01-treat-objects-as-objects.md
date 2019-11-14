This may seem obvious; you should generally  give separate objects in your problem domain a special class in your  code. We've seen examples of this in the case studies in previous  chapters: first, we identify objects in the problem, and then model  their data and behaviors.

Identifying objects is a very important  task in object-oriented analysis and programming. But it isn't always as  easy as counting the nouns in short paragraphs that, frankly, I have  constructed explicitly for that purpose. Remember, objects are things  that have both data and behavior. If we are working only with data, we  are often better off storing it in a list, set, dictionary, or other  Python data structure (which we'll be covering thoroughly in **Python Data Structures**). On the other hand, if we are working only with behavior, but no stored data, a simple function is more suitable.

An  object, however, has both data and behavior. Proficient Python  programmers use built-in data structures unless (or until) there is an  obvious need to define a class. There is no reason to add an extra level  of abstraction if it doesn't help organize our code. On the other hand,  the **obvious** need is not always self-evident.

We  can often start our Python programs by storing data in a few variables.  As the program expands, we will later find that we are passing the same  set of related variables to a set of functions. This is the time to  think about grouping both variables and functions into a class. If we  are designing a program to model polygons in two-dimensional space, we  might start with each polygon represented as a list of points. The  points would be modeled as two tuples (**x**, **y**)  describing where that point is located. This is all data, stored in a  set of nested data structures (specifically, a list of tuples):

```python
square = [(1,1), (1,2), (2,2), (2,1)] 
```

Now, if we want to calculate the distance  around the perimeter of the polygon, we need to sum the distances  between each point. To do this, we need a function to calculate the  distance between two points. Here are two such functions:

```python
import math

def distance(p1, p2):
    return math.sqrt((p1[0]-p2[0])**2 + (p1[1]-p2[1])**2)

def perimeter(polygon):
    perimeter = 0
    points = polygon + [polygon[0]]
    for i in range(len(polygon)):
        perimeter += distance(points[i], points[i+1])
    return perimeter
```

Now, as object-oriented programmers, we clearly recognize that a `polygon` class could encapsulate the list of points (data) and the `perimeter` function (behavior). Further, a `point` class, such as we defined in **Objects in Python** might encapsulate the `x` and `y` coordinates and the `distance` method. The question is: is it valuable to do this?

For  the previous code, maybe yes, maybe no. With our recent experience in  object-oriented principles, we can write an object-oriented version in  record time. Let's compare them as follows:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def distance(self, p2):
        return math.sqrt((self.x-p2.x)**2 + (self.y-p2.y)**2)

class Polygon:
    def __init__(self):
        self.vertices = []

    def add_point(self, point):
        self.vertices.append((point))

    def perimeter(self):
        perimeter = 0
        points = self.vertices + [self.vertices[0]]
        for i in range(len(self.vertices)):
            perimeter += points[i].distance(points[i+1])
        return perimeter
```

As we can see from the highlighted sections, there is twice as much code here as there was in our earlier version, although we could argue that the `add_point` method is not strictly necessary.

Now,  to understand the differences a little better, let's compare the two  APIs in use. Here's how to calculate the perimeter of a square using the  object-oriented code:

```python
>>> square = Polygon()>>> square.add_point(Point(1,1))>>> square.add_point(Point(1,2))>>> square.add_point(Point(2,2))>>> square.add_point(Point(2,1))>>> square.perimeter()4.0
```

That's fairly succinct and easy to read, you might think, but let's compare it to the function-based code:

```python
>>> square = [(1,1), (1,2), (2,2), (2,1)]>>> perimeter(square)4.0
```

Hmm, maybe the object-oriented API isn't so compact! That said, I'd argue that it was easier to **read**  than the functional example. How do we know what the list of tuples is  supposed to represent in the second version? How do we remember what  kind of object we're supposed to pass into the `perimeter`  function? (a list of two tuples? That's not intuitive!) We would need a  lot of documentation to explain how these functions should be used.

In  contrast, the object-oriented code is relatively self-documenting. We  just have to look at the list of methods and their parameters to know  what the object does and how to use it. By the time we wrote all the  documentation for the functional version, it would probably be longer  than the object-oriented code.

Finally, code length is not a good indicator of code complexity. Some programmers get hung up on complicated **one liners** that  do an incredible amount of work in one line of code. This can be a fun  exercise, but the result is often unreadable, even to the original  author the following day. Minimizing the amount of code can often make a  program easier to read, but do not blindly assume this is the case.

Luckily, this trade-off isn't necessary. We can make the object-oriented `Polygon` API as easy to use as the functional implementation. All we have to do is alter our `Polygon` class so that it can be constructed with multiple points. Let's give it an initializer that accepts a list of `Point` objects. In fact, let's allow it to accept tuples too, and we can construct the `Point` objects ourselves, if needed:

```python
def __init__(self, points=None): 
    points = points if points else [] 
    self.vertices = [] 
    for point in points: 
        if isinstance(point, tuple): 
            point = Point(*point) 
        self.vertices.append(point) 
```

This  initializer goes through the list and ensures that any tuples are  converted to points. If the object is not a tuple, we leave it as is,  assuming that it is either a `Point` object already, or an unknown duck-typed object that can act like a `Point` object.

info> If you are experimenting with the above code, you could subclass `Polygon` and override the `__init__` function instead of replacing the initializer or copying the `add_point` and `perimeter` methods.

Still,  there's no clear winner between the object-oriented and more  data-oriented versions of this code. They both do the same thing. If we  have new functions that accept a polygon argument, such as `area(polygon)` or `point_in_polygon(polygon, x, y)`, the benefits of the object-oriented code become increasingly obvious. Likewise, if we add other attributes to the polygon, such as `color` or `texture`, it makes more and more sense to encapsulate that data into a single class.

The  distinction is a design decision, but in general, the more important a  set of data is, the more likely it is to have multiple functions  specific to that data, and the more useful it is to use a class with  attributes and methods instead.

When making this decision, it also  pays to consider how the class will be used. If we're only trying to  calculate the perimeter of one polygon in the context of a much greater  problem, using a function will probably be quickest to code and easier  to use **one time only**. On the  other hand, if our program needs to manipulate numerous polygons in a  wide variety of ways (calculating the perimeter, area, and intersection  with other polygons, moving or scaling them, and so on), we have almost  certainly identified an object; one that needs to be extremely  versatile.

Additionally, pay attention to the interaction between  objects. Look for inheritance relationships; inheritance is impossible  to model elegantly without classes, so make sure to use them. Look for  the other types of relationships we discussed in **Object-Oriented Design**  association and composition. Composition can, technically, be modeled  using only data structures; for example, we can have a list of  dictionaries holding tuple values, but it is sometimes less complicated  to create a few classes of objects, especially if there is behavior associated with the data.

info> Don't  rush to use an object just because you can use an object, but don't  neglect to create a class when you need to use a class.