We now have a grasp of some basic object-oriented terminology. Objects are instances of classes that can be associated with each other. An object instance  is a specific object with its own set of data and behaviors; a specific  orange on the table in front of us is said to be an instance of the  general class of oranges. That's simple enough, but let's dive into the  meaning of those two words, **data** and **behaviors**.

# Data Describes Objects

Let's start with data. Data represents the individual characteristics of a certain  object. A class can define specific sets of characteristics that are  shared by all objects from that class. Any specific object can have  different data values for the given characteristics. For example, the  three oranges on our table (if we haven't eaten any) could each weigh a  different amount. The orange class could have a weight attribute to  represent that datum. All instances of the orange class have a weight  attribute, but each orange has a different value for this attribute.  Attributes don't have to be unique, though; any two oranges may weigh  the same amount. As a more realistic example, two objects representing  different customers might have the same value for a first name  attribute.

Attributes are frequently referred to as **members** or **properties**. Some authors  suggest that the terms have different meanings, usually that attributes  are settable, while properties are read-only. In Python, the concept of  **read-only** is rather pointless, so throughout this book, we'll see the two terms used interchangeably. In addition, as we'll discuss in **When to Use Object-Oriented Programming**, the `property` keyword has a special meaning in Python for a particular kind of attribute.

In  our fruit inventory application, the fruit farmer may want to know what  orchard the orange came from, when it was picked, and how much it  weighs. They might also want to keep track of where each **Basket**  is stored. Apples might have a color attribute, and barrels might come  in different sizes. Some of these properties may also belong to multiple  classes (we may want to know when apples are picked, too), but for this  first example, let's just add a few different attributes to our class  diagram:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/0c8aeb83-f802-40f8-acec-9314a7c52eef.png)

Depending on how detailed our design needs to be, we can also specify the type for each attribute. Attribute types are often primitives that are standard to most programming languages, such as integer, floating-point number, string, byte, or Boolean. However, they can also represent data structures such as lists, trees, or graphs, or most notably, other classes. This is one area where the design stage can overlap with the programming stage. The various primitives or objects available in one programming language may be different from what is available in another:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/dcd02f07-d516-4e2a-9d27-bbd92bcc31f7.png)

Usually, we don't need to be overly concerned with data types at the design stage, as implementation-specific details are chosen during the programming stage. Generic names are normally sufficient for design. If our design calls for a list container type, Java programmers can choose to use a `LinkedList` or an `ArrayList` when implementing it, while Python programmers (that's us!) might choose between the `list` built-in and a `tuple`.

In  our fruit-farming example so far, our attributes are all basic  primitives. However, there are some implicit attributes that we can make  explicit—the associations. For a given orange, we might have an  attribute referring to the basket that holds that orange.

# Behaviors are Actions

Now that we know what data is, the last undefined term is **behaviors**. Behaviors are actions that can occur on an object. The behaviors that can be performed on a specific class of object are called **methods**. At the programming level, methods are like functions in structured programming, but they **magically** have access to all the data associated with this object. Like functions, methods can also accept **parameters** and return **values**.

A method's parameters are provided to it as a list of objects that need to be **passed** into that method. The actual object instances that are passed into a method during a specific invocation are usually referred to as **arguments**.  These objects are used by the method to perform whatever behavior or  task it is meant to do. Returned values are the results of that task.

We've stretched our **comparing apples and oranges** example  into a basic (if far-fetched) inventory application. Let's stretch it a  little further and see whether it breaks. One action that can be  associated with oranges is the **pick** action. If you think about implementation, **pick** would need to do two things:

- Place the orange in a basket by updating the **Basket** attribute of the orange
- Add the orange to the **Orange** list on the given **Basket**.

So, **pick** needs to know what basket it is dealing with. We do this by giving the **pick** method a **Basket** parameter. Since our fruit farmer also sells juice, we can add a **squeeze** method to the **Orange** class. When called, the **squeeze** method might return the amount of juice retrieved, while also removing the **Orange** from the **Basket** it was in.

The class **Basket** can have a **sell** action. When a basket is sold, our inventory system might update some data on as-yet unspecified objects for accounting and profit calculations. Alternatively, our basket of oranges might go bad before we can sell them, so we add a **discard** method. Let's add these methods to our diagram:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/1ab4c873-4cfa-427d-8e53-b58a2b8df264.png)

Adding attributes and methods to individual objects allows us to create a **system**  of interacting objects. Each object in the system is a member of a  certain class. These classes specify what types of data the object can  hold and what methods can be invoked on it. The data in each object can  be in a different state from other instances of the same class; each  object may react to method calls differently because of the differences  in state.

Object-oriented analysis and design is all about figuring out what those objects  are and how they should interact. The next section describes principles  that can be used to make those interactions as simple and intuitive as  possible.