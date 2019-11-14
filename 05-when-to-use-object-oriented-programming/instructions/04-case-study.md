For this case study, we'll try to delve further into the question,  When should I choose an object versus a built-in type? We'll be modeling  a `Document` class that might be used in a text editor or word processor. What objects, functions, or properties should it have?

We might start with a `str` for the `Document` contents, but in Python, strings aren't mutable (able to be changed). Once a `str`  is defined, it is forever. We can't insert a character into it or  remove one without creating a brand new string object. That would be  leaving a lot of `str` objects taking up memory until Python's garbage collector sees fit to clean up behind us.

So,  instead of a string, we'll use a list of characters, which we can  modify at will. In addition, we'll need to know the current cursor  position within the list, and should probably also store a filename for  the document.

info> Real text editors use a binary tree-based data structure called a `rope` to model their document contents. This book's title isn't **Advanced Data Structures**, so if you're interested in learning more about this fascinating topic, you may want to search the web for **rope data structure**.

There  are a lot of things we might want to do to a text document, including  inserting, deleting, and selecting characters; cutting, copying, and,  pasting the selection; and saving or closing the document. It looks like  there are copious amounts of both data and behavior, so it makes sense to put all this stuff into its own `Document` class.

A pertinent question is: should this class be composed of a bunch of basic Python objects such as `str` filenames, `int` cursor positions, and a `list`  of characters? Or should some or all of those things be specially  defined objects in their own right? What about individual lines and  characters? Do they need to have classes of their own?

We'll answer these questions as we go, but let's start with the simplest possible class first- `Document`  and see what it can do:

```python
class Document: 
    def __init__(self): 
        self.characters = [] 
        self.cursor = 0 
        self.filename = '' 
 
    def insert(self, character): 
        self.characters.insert(self.cursor, character) 
        self.cursor += 1 
 
    def delete(self): 
        del self.characters[self.cursor] 
 
    def save(self): 
        with open(self.filename, 'w') as f: 
            f.write(''.join(self.characters)) 
 
    def forward(self): 
        self.cursor += 1 
 
    def back(self): 
        self.cursor -= 1 
```

This basic class allows us full control over editing a basic document. Have a look at it in action:

```python
>>> doc = Document()>>> doc.filename = "test_document">>> doc.insert('h')>>> doc.insert('e')>>> doc.insert('l')>>> doc.insert('l')>>> doc.insert('o')>>> "".join(doc.characters)'hello'>>> doc.back()>>> doc.delete()>>> doc.insert('p')>>> "".join(doc.characters)'hellp'
```

It  looks like it's working. We could connect a keyboard's letter and arrow  keys to these methods and the document would track everything just  fine.

But what if we want to connect more than just arrow keys. What if we want to connect the ***Home*** and ***End*** keys as well? We could add more methods to the `Document` class that search forward or backward for newline characters (a newline character, escaped as `\n`,  represents the end of one line and the beginning of a new one) in the  string and jump to them, but if we did that for every possible movement  action (move by words, move by sentences, **Page Up**, **Page Down**,  end of line, beginning of white space, and others), the class would be  huge. Maybe it would be better to put those methods on a separate  object. So, let's turn the `Cursor` attribute into an object that is aware of its position and can manipulate that position. We can move the forward and back methods to that class, and add a couple more for the **`Home`**` and `**`End`** keys, as follows:

```python
class Cursor:
    def __init__(self, document):
        self.document = document
        self.position = 0

    def forward(self):
        self.position += 1

    def back(self):
        self.position -= 1

    def home(self):
        while self.document.characters[self.position - 1].character != "\n":
            self.position -= 1
            if self.position == 0:
                # Got to beginning of file before newline
                break

    def end(self):
        while (
            self.position < len(self.document.characters)
            and self.document.characters[self.position] != "\n"
        ):
            self.position += 1
```

This class takes the  document as an initialization parameter so the methods have access to  the content of the document's character list. It then provides simple  methods for moving backward and forward, as before, and for moving to the `home` and `end` positions.

info> This  code is not very safe. You can very easily move past the ending  position, and if you try to go home on an empty file, it will crash.  These examples are kept short to make them readable, but that doesn't  mean they are defensive! You can improve the error checking of this code  as an exercise; it might be a great opportunity to expand your  exception-handling skills.

The `Document` class itself is hardly changed, except for removing the two methods that were moved to the `Cursor` class:

```python
class Document: 
    def __init__(self): 
        self.characters = [] 
        self.cursor = Cursor(self) 
        self.filename = '' 
 
       def insert(self, character): 
        self.characters.insert(self.cursor.position, 
                character) 
        self.cursor.forward() 
 
    def delete(self): 
        del self.characters[self.cursor.position] 
 
    def save(self):
        with open(self.filename, "w") as f:
            f.write("".join(self.characters))
```

We just updated anything that accessed the old cursor integer to use the new object instead. We can now test that the `home` method is really moving to the newline character, as follows:

```python
>>> d = Document()>>> d.insert('h')>>> d.insert('e')>>> d.insert('l')>>> d.insert('l')>>> d.insert('o')>>> d.insert('\n')>>> d.insert('w')>>> d.insert('o')>>> d.insert('r')>>> d.insert('l')>>> d.insert('d')>>> d.cursor.home()>>> d.insert("*")>>> print("".join(d.characters))hello*world
```

Now, since we've been using that string `join` function a lot (to concatenate the characters so we can see the actual document contents), we can add a property to the `Document` class to give us the complete string as follows:

```python
@property 
def string(self): 
    return "".join(self.characters) 
```

This makes our testing a little simpler:

```python
>>> print(d.string)helloworld
```

This  framework is simple to extend, create and edit a complete plain text  document (though it might be a bit time consuming!) Now, let's extend it  to work for rich text; text that can have **bold**, underlined, or **italic** characters.

There are two ways we could process this. The first is to insert **fake** characters into our character list that act like instructions, such as **bold characters until you find a stop bold character**.  The second is to add information to each character, indicating what  formatting it should have. While the former method is more common in  real editors, we'll implement the latter solution. To do that, we're  obviously going to need a class for characters. This class will have an  attribute representing the character, as well as three Boolean  attributes representing whether it is **bold, italic, or underlined**.

Hmm, wait! Is this `Character`  class going to have any methods? If not, maybe we should use one of the  many Python data structures instead; a tuple or named tuple would  probably be sufficient. Are there any actions that we would want to  execute or invoke on a character?

Well, clearly, we might want to  do things with characters, such as delete or copy them, but those are  things that need to be handled at the `Document` level, since they are really modifying the list of characters. Are there things that need to be done to individual characters?

Actually, now that we're thinking about what a `Character` class actually **is**... what is it? Would it be safe to say that a `Character`  class is a string? Maybe we should use an inheritance relationship  here? Then we can take advantage of the numerous methods that `str` instances come with.

What sorts of methods are we talking about? There's `startswith`, `strip`, `find`, `lower`, and many more. Most of these methods expect to be working on strings that contain more than one character. In contrast, if `Character` were to subclass `str`, we'd probably be wise to override `__init__`  to raise an exception if a multi-character string were supplied. Since  all those methods we'd get for free wouldn't really apply to our `Character` class, it seems we shouldn't use inheritance, after all.

This brings us back to our original question; should `Character` even be a class? There is a very important special method on the `object` class that we can take advantage of to represent our characters. This method, called `__str__` (two underscores at each end, like `__init__`), is used in string-manipulation functions such as `print` and the `str`  constructor to convert any class to a string. The default  implementation does some boring stuff, such as printing the name of the  module and class, and its address in memory. But if we override it, we  can make it print whatever we like. For our implementation, we could  make it prefix characters with special characters to represent whether  they are bold, italic, or underlined. So, we will create a class to  represent a character, and here it is:

```python
class Character: 
    def __init__(self, character, 
            bold=False, italic=False, underline=False): 
        assert len(character) == 1 
        self.character = character 
        self.bold = bold 
        self.italic = italic 
        self.underline = underline 
 
    def __str__(self): 
        bold = "*" if self.bold else '' 
        italic = "/" if self.italic else '' 
        underline = "_" if self.underline else '' 
        return bold + italic + underline + self.character 
```

This class allows us to create characters and prefix them with a special character when the `str()` function is applied to them. Nothing too exciting there. We only have to make a few minor modifications to the `Document` and `Cursor` classes to work with this class. In the `Document` class, we add these two lines at the beginning of the `insert` method, as follows:

```python
def insert(self, character): 
    if not hasattr(character, 'character'): 
        character = Character(character) 
```

This is a rather strange bit of code. Its basic purpose is to check whether the character being passed in is a `Character` or a `str`. If it is a string, it is wrapped in a `Character` class so all objects in the list are `Character` objects. However, it is entirely possible that someone using our code would want to use a class that is neither a `Character` nor a string, using duck typing. If the object has a character attribute, we assume it is a `Character`-like object. But if it does not, we assume it is a `str`-like object and wrap it in `Character`.  This helps the program take advantage of duck typing as well as  polymorphism; as long as an object has a character attribute, it can be  used in the `Document` class.

This  generic check could be very useful. For example, if we wanted to make a  programmer's editor with syntax highlighting, we'd need extra data on  the character, such as what type of syntax token the character belongs  to. Note that, if we are doing a lot of this kind of comparison, it's  probably better to implement `Character` as an abstract base class with an appropriate `__subclasshook__`, as discussed in **When Objects Are Alike**.

In addition, we need to modify the string property on `Document` to accept the new `Character` values. All we need to do is call `str()` on each character before we join it, as demonstrated in the following:

```python
    @property 
    def string(self): 
        return "".join((str(c) for c in self.characters)) 
```

This code uses a generator expression, which we'll discuss in **The Iterator Pattern**. It's a shortcut to perform a specific action on all the objects in a sequence.

Finally, we also need to check `Character.character`, instead of just the string character we were storing before, in the `home` and `end` functions when we're looking to see whether it matches a newline character, as demonstrated in the following:

```python
    def home(self): 
        while self.document.characters[ 
                self.position-1].character != '\n': 
            self.position -= 1 
            if self.position == 0: 
                # Got to beginning of file before newline 
                break 
 
    def end(self): 
        while self.position < len( 
                self.document.characters) and \ 
                self.document.characters[ 
                        self.position 
                        ].character != '\n': 
            self.position += 1 
```

This completes the formatting of characters. We can test it to see that it works as follows:

```python
>>> d = Document()>>> d.insert('h')>>> d.insert('e')>>> d.insert(Character('l', bold=True))>>> d.insert(Character('l', bold=True))>>> d.insert('o')>>> d.insert('\n')>>> d.insert(Character('w', italic=True))>>> d.insert(Character('o', italic=True))>>> d.insert(Character('r', underline=True))>>> d.insert('l')>>> d.insert('d')>>> print(d.string)he*l*lo/w/o_rld>>> d.cursor.home()>>> d.delete()>>> d.insert('W')>>> print(d.string)he*l*loW/o_rld>>> d.characters[0].underline = True>>> print(d.string)_he*l*loW/o_rld
```

As expected, whenever we print the string, each bold character is preceded by a `*` character, each italicized character by a `/` character, and each underlined character by a `_`  character. All our functions seem to work, and we can modify characters  in the list after the fact. We have a working rich text document object  that could be plugged into a proper graphical user interface and hooked  up with a keyboard for input and a screen for output. Naturally, we'd  want to display real **bold, italic, and underlined** fonts in a UI, instead of using our `__str__` method, but it was sufficient for the basic testing we demanded of it.