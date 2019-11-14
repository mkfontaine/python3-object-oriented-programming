The command pattern adds a level of abstraction between actions that  must be done and the object that invokes those actions, normally at a  later time. In the command pattern, client code creates a `Command`  object that can be executed at a later date. This object knows about a  receiver object that manages its own internal state when the command is  executed on it. The `Command` object implements a specific interface (typically, it has an `execute` or `do_action` method, and also keeps track of any arguments required to perform the action. Finally, one or more `Invoker` objects execute the command at the correct time.

Here's the UML diagram:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/1046b535-724f-4129-b809-ee86d21ad64a.png)

A common example of the command pattern is actions on a graphical  window. Often, an action can be invoked by a menu item on the menu bar, a  keyboard shortcut, a toolbar icon, or a context menu. These are all  examples of `Invoker` objects. The actions that actually occur, such as `Exit`, `Save`, or `Copy`, are implementations of `CommandInterface`. A GUI window to receive exit, a document to receive save, and `ClipboardManager` to receive copy commands, are all examples of possible `Receivers`.

Let's implement a simple command pattern that provides commands for `Save` and `Exit` actions. We'll start with some modest receiver classes, themselves with the following code:

```python
import sys 
 
class Window: 
    def exit(self): 
        sys.exit(0) 
 
class Document: 
    def __init__(self, filename): 
        self.filename = filename 
        self.contents = "This file cannot be modified" 
 
    def save(self): 
        with open(self.filename, 'w') as file: 
            file.write(self.contents) 
```

These mock  classes model objects that would likely be doing a lot more in a working  environment. The window would need to handle mouse movement and  keyboard events, and the document would need to handle character  insertion, deletion, and selection. But for our example, these two  classes will do what we need.

Now let's define some invoker  classes. These will model toolbar, menu, and keyboard events that can  happen; again, they aren't actually hooked up to anything, but we can  see how they are decoupled from the command, receiver, and client code  in the following code snippet:

```python
class ToolbarButton:
    def __init__(self, name, iconname):
        self.name = name
        self.iconname = iconname

    def click(self):
        self.command.execute()


class MenuItem:
    def __init__(self, menu_name, menuitem_name):
        self.menu = menu_name
        self.item = menuitem_name

    def click(self):
        self.command.execute()


class KeyboardShortcut:
    def __init__(self, key, modifier):
        self.key = key
        self.modifier = modifier

    def keypress(self):
        self.command.execute()
```

Notice how the various action methods each call the `execute` method on their respective commands? This code doesn't show the `command` attribute being set on each object. They could be passed into the `__init__`  function, but because they may be changed (for example, with a  customizable keybinding editor), it makes more sense to set the  attributes on the objects afterwards.

Now, let's hook up the commands themselves with the following code:

```python
class SaveCommand:
    def __init__(self, document):
        self.document = document

    def execute(self):
        self.document.save()


class ExitCommand:
    def __init__(self, window):
        self.window = window

    def execute(self):
        self.window.exit()
```

These commands are  straightforward; they demonstrate the basic pattern, but it is important  to note that we can store state and other information with the command  if necessary. For example, if we had a command to insert a character, we  could maintain state for the character currently being inserted.

Now  all we have to do is hook up some client and test code to make the  commands work. For basic testing, we can just include the following code  at the end of the script:

```python
window = Window() 
document = Document("a_document.txt") 
save = SaveCommand(document) 
exit = ExitCommand(window) 
 
save_button = ToolbarButton('save', 'save.png') 
save_button.command = save 
save_keystroke = KeyboardShortcut("s", "ctrl") 
save_keystroke.command = save 
exit_menu = MenuItem("File", "Exit") 
exit_menu.command = exit 
```

First, we create two  receivers and two commands. Then, we create several of the available  invokers and set the correct command on each of them. To test, we can  use `python3``-i``filename.py` and run code such as `exit_menu.click()`, which will end the program, or `save_keystroke.keystroke()`, which will save the fake file.

Unfortunately,  the preceding examples do not feel terribly Pythonic. They have a lot  of "boilerplate code" (code that does not accomplish anything, but only  provides structure to the pattern), and the `Command`  classes are all eerily similar to each other. Perhaps we could create a  generic command object that takes a function as a callback?

In fact, why bother? Can we just use a function or method object for each command? Instead of an object with an `execute()`  method, we can write a function and use that as the command directly.  The following is a common paradigm for the command pattern in Python:

```python
import sys


class Window:
    def exit(self):
        sys.exit(0)


class MenuItem:
    def click(self):
        self.command()


window = Window()
menu_item = MenuItem()
menu_item.command = window.exit
```

Now that looks a  lot more like Python. At first glance, it looks like we've removed the  command pattern altogether, and we've tightly connected the `menu_item` and `Window` classes. But if we look closer, we find there is no tight coupling at all. Any callable can be set up as the command on `MenuItem`, just as before. And the `Window.exit`  method can be attached to any invoker. Most of the flexibility of the  command pattern has been maintained. We have sacrificed complete  decoupling for readability, but this code is, in my opinion, and that of  many Python programmers, more maintainable than the fully abstracted  version.

Of course, since we can add a `__call__`  method to any object, we aren't restricted to functions. The previous  example is a useful shortcut when the method being called doesn't have  to maintain state, but in more advanced usage, we can use the following  code as well:

```python
class Document:
    def __init__(self, filename):
        self.filename = filename
        self.contents = "This file cannot be modified"

    def save(self):
        with open(self.filename, "w") as file:
            file.write(self.contents)


class KeyboardShortcut:
    def keypress(self):
        self.command()


class SaveCommand:
    def __init__(self, document):
        self.document = document

    def __call__(self):
        self.document.save()


document = Document("a_file.txt")
shortcut = KeyboardShortcut()
save_command = SaveCommand(document)
shortcut.command = save_command
```

Here, we have  something that looks like the first command pattern, but a bit more  idiomatic. As you can see, making the invoker call a callable instead of  a `command` object with an execute method  has not restricted us in any way. In fact, it's given us more  flexibility. We can link to functions directly when that works, yet we  can build a complete callable `command` object when the situation calls for it.

The command pattern is often extended to support undoable commands. For example, a text program may wrap each insertion in a separate command with not only an `execute` method, but also an `undo`  method that will delete that insertion. A graphics program may wrap  each drawing action (rectangle, line, freehand pixels, and so on) in a  command that has an `undo` method that resets  the pixels to their original state. In such cases, the decoupling of  the command pattern is much more obviously useful, because each action  has to maintain enough of its state to undo that action at a later date.