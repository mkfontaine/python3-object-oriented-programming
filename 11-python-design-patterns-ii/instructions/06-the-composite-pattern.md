The composite pattern allows  complex tree-like structures to be built from simple components. These  components, called composite objects, are able to behave sort of like a  container and sort of like a variable, depending on whether they have  child components. Composite objects are container objects, where the  content may actually be another composite object.

Traditionally,  each component in a composite object must be either a leaf node (that  cannot contain other objects) or a composite node. The key is that both  composite and leaf nodes can have the same interface. The following UML  diagram is very simple:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/57741dbb-c73e-4262-ae38-0c3b0e4994a8.png)

This simple pattern, however, allows us to create complex arrangements of elements, all of which satisfy the interface of the component object. The following diagram depicts a concrete instance of such a complicated arrangement:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/5c7c575c-4c2c-4736-a898-ad6a24afa790.png)

The composite pattern is commonly useful in file/folder-like trees.  Regardless of whether a node in the tree is a normal file or a folder,  it is still subject to operations such as moving, copying, or deleting  the node. We can create a component interface that supports these  operations, and then use a composite object to represent folders, and  leaf nodes to represent normal files.

Of course, in Python, once  again, we can take advantage of duck typing to implicitly provide the  interface, so we only need to write two classes. Let's define these  interfaces first in the following code:

```python
class Folder: 
    def __init__(self, name): 
        self.name = name 
        self.children = {} 
 
    def add_child(self, child): 
        pass 
 
    def move(self, new_path): 
        pass 
 
    def copy(self, new_path): 
        pass 
 
    def delete(self): 
        pass 
 
class File: 
    def __init__(self, name, contents): 
        self.name = name 
        self.contents = contents 
 
    def move(self, new_path): 
        pass 
 
    def copy(self, new_path): 
        pass 
 
    def delete(self): 
        pass 
```

For each folder (composite) object,  we maintain a dictionary of children. For many composite  implementations, a list is sufficient, but in this case, a dictionary  will be useful for looking up children by name. Our paths will be  specified as node names separated by the `/` character, similar to paths in a Unix shell.

Thinking about the methods involved, we can see that  moving or deleting a node behaves in a similar way, regardless of  whether or not it is a file or folder node. Copying, however, has to do a  recursive copy for folder nodes, while copying a file node is a trivial  operation.

To take advantage of the similar operations, we can  extract some of the common methods into a parent class. Let's take that  discarded `Component` interface and change it to a base class with the following code:

```python
class Component:
    def __init__(self, name):
        self.name = name

    def move(self, new_path):
        new_folder = get_path(new_path)
        del self.parent.children[self.name]
        new_folder.children[self.name] = self
        self.parent = new_folder

    def delete(self):
        del self.parent.children[self.name]


class Folder(Component):
    def __init__(self, name):
        super().__init__(name)
        self.children = {}

    def add_child(self, child):
        pass

    def copy(self, new_path):
        pass


class File(Component):
    def __init__(self, name, contents):
        super().__init__(name)
        self.contents = contents

    def copy(self, new_path):
        pass


root = Folder("")


def get_path(path):
    names = path.split("/")[1:]
    node = root
    for name in names:
        node = node.children[name]
    return node
```

We've created the `move` and `delete` methods on the `Component` class. Both of them access a mysterious `parent` variable that we haven't set yet. The `move` method uses a module-level `get_path`  function that finds a node from a predefined root node, given a path.  All files will be added to this root node or a child of that node. For  the `move` method, the target should be an  existing folder, or we'll get an error. As in many examples in technical  books, error handling is woefully absent, to help focus on the  principles under consideration.

Let's set up that mysterious `parent` variable in the folder's `add_child` method, as follows:

```python
    def add_child(self, child):
        child.parent = self
        self.children[child.name] = child
```

Well, that was easy enough. Let's see if our composite file hierarchy is working properly with the following code snippet:

```bash
$ python3 -i 1261_09_18_add_child.py>>> folder1 = Folder('folder1')>>> folder2 = Folder('folder2')>>> root.add_child(folder1)>>> root.add_child(folder2)>>> folder11 = Folder('folder11')>>> folder1.add_child(folder11)>>> file111 = File('file111', 'contents')>>> folder11.add_child(file111)>>> file21 = File('file21', 'other contents')>>> folder2.add_child(file21)>>> folder2.children{'file21': <__main__.File object at 0xb7220a4c>}>>> folder2.move('/folder1/folder11')>>> folder11.children{'folder2': <__main__.Folder object at 0xb722080c>, 'file111': <__main__.File object at 
0xb72209ec>}>>> file21.move('/folder1')>>> folder1.children{'file21': <__main__.File object at 0xb7220a4c>, 'folder11': <__main__.Folder object at 
0xb722084c>}
```

Yes, we can create  folders, add folders to other folders, add files to folders, and move  them around! What more could we ask for in a file hierarchy?

Well, we could ask for copying to be implemented, but to conserve trees, let's leave that as an exercise.

The  composite pattern is extremely useful for a variety of tree-like  structures, including GUI widget hierarchies, file hierarchies, tree  sets, graphs, and HTML DOM. It can be a useful pattern in Python when  implemented according to the traditional implementation, as in the  example demonstrated earlier. Sometimes, if only a shallow tree is being  created, we can get away with a list of lists or a dictionary  of dictionaries, and do not need to implement custom component, leaf,  and composite classes. Other times, we can get away with implementing  only one composite class, and treating leaf and composite objects as a  single class. Alternatively, Python's duck typing can make it easy to  add other objects to a composite hierarchy, as long as they have the  correct interface.