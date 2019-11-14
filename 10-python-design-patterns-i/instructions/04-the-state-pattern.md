The state pattern is structurally similar  to the strategy pattern, but its intent and purpose are very different.  The goal of the state pattern is to represent state-transition systems:  systems where it is obvious that an object can be in a specific state,  and that certain activities may drive it to a different state.

To  make this work, we need a manager, or context class that provides an  interface for switching states. Internally, this class contains a  pointer to the current state. Each state knows what other states it is  allowed to be in and will transition to those states depending on  actions invoked upon it.

So, we have two types of classes: the  context class and multiple state classes. The context class maintains  the current state, and forwards actions to the state classes. The state  classes are typically hidden from any other objects that are calling the  context; it acts like a black box that happens to perform state  management internally. Here's how it looks in UML:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/d8d33bf6-2f16-44cd-bdc9-c640b5175520.png)

# A State Example

To  illustrate the state pattern, let's build an XML parsing tool. The  context class will be the parser itself. It will take a string as input  and place the tool in an initial parsing state. The various parsing  states will eat characters, looking for a specific value, and when that value  is found, change to a different state. The goal is to create a tree of  node objects for each tag and its contents. To keep things manageable,  we'll parse only a subset of XML – tags and tag names. We won't be able  to handle attributes on tags. It will parse text content of tags, but  won't attempt to parse **mixed** content, which has tags inside of text. Here is an example **simplified XML** file that we'll be able to parse:

```xml
<book> 
    <author>Dusty Phillips</author> 
    <publisher>Packt Publishing</publisher> 
    <title>Python 3 Object Oriented Programming</title> 
    <content> 
        <chapter> 
            <number>1</number> 
            <title>Object Oriented Design</title> 
        </chapter> 
        <chapter> 
            <number>2</number> 
            <title>Objects In Python</title> 
        </chapter> 
    </content> 
</book> 
```

Before we look at the states and the parser, let's consider the output of this program. We know we want a tree of `Node` objects, but what does a `Node`  look like? It will clearly need to know the name of the tag it is  parsing, and since it's a tree, it should probably maintain a pointer to  the parent node and a list of the node's children in order. Some nodes  have a text value, but not all of them. Let's look at this `Node` class first:

```python
class Node:
    def __init__(self, tag_name, parent=None):
        self.parent = parent
        self.tag_name = tag_name
        self.children = []
        self.text = ""

    def __str__(self):
        if self.text:
            return self.tag_name + ": " + self.text
        else:
            return self.tag_name
```

This class sets default attribute values upon initialization. The `__str__` method is supplied to help visualize the tree structure when we're finished.

Now,  looking at the example document, we need to consider what states our  parser can be in. Clearly, it's going to start in a state where no nodes  have yet been processed. We'll need a state for processing opening tags  and closing tags. And when we're inside a tag with text contents, we'll  have to process that as a separate state, too.

Switching states  can be tricky; how do we know if the next node is an opening tag, a  closing tag, or a text node? We could put a little logic in each state  to work this out, but it actually makes more sense to create a new state  whose sole purpose is figuring out which state we'll be switching to  next. If we call this transition state **ChildNode**, we end up with the following states:

- `FirstTag`
- `ChildNode`
- `OpenTag`
- `CloseTag`
- `Text`

The **FirstTag** state will switch to **ChildNode**,  which is responsible for deciding which of the other three states to  switch to; when those states are finished, they'll switch back to **ChildNode**. The following state-transition diagram shows the available state changes:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/7600bf5b-6e08-41dc-89a1-77bff8e58fda.png)

The states are responsible for taking **what's left of the string**,  processing as much of it as they know what to do with, and then telling  the parser to take care of the rest of it. Let's construct the `Parser` class first:

```python
class Parser: 
    def __init__(self, parse_string): 
        self.parse_string = parse_string 
        self.root = None 
        self.current_node = None 
 
        self.state = FirstTag() 
 
    def process(self, remaining_string): 
        remaining = self.state.process(remaining_string, self) 
        if remaining: 
            self.process(remaining) 
 
    def start(self): 
        self.process(self.parse_string) 
```

The initializer sets up a few variables on the class that the individual states will access. The `parse_string` instance variable is the text that we are trying to parse. The `root` node is the **top** node in the XML structure. The `current_node` instance variable is the one that we are currently adding children to.

The important feature of this parser is the `process` method, which accepts the remaining string, and passes it off to the current state. The parser (the `self` argument) is also passed into the state's process method so that the state can manipulate it. The state is expected to return the remainder of the unparsed string when it is finished processing. The parser then recursively calls the `process` method on this remaining string to construct the rest of the tree.

Now let's have a look at the `FirstTag` state:

```python
class FirstTag:
    def process(self, remaining_string, parser):
        i_start_tag = remaining_string.find("<")
        i_end_tag = remaining_string.find(">")
        tag_name = remaining_string[i_start_tag + 1 : i_end_tag]
        root = Node(tag_name)
        parser.root = parser.current_node = root
        parser.state = ChildNode()
        return remaining_string[i_end_tag + 1 :]
```

This state finds the index (the `i_`  stands for index) of the opening and closing angle brackets on the  first tag. You may think this state is unnecessary, since XML requires  that there be no text before an opening tag. However, there may be  whitespace that needs to be consumed; this is why we search for the  opening angle bracket instead of assuming it is the first character in  the document.

info> Note  that this code is assuming a valid input file. A proper implementation  would be rigorously testing for invalid input, and would attempt to  recover or display an extremely descriptive error message.

The method extracts the name of the tag and assigns it to the root node of the parser. It also assigns it to `current_node`, since that's the one we'll be adding children to next.

Then comes the important part: the method changes the current state on the parser object to a `ChildNode` state. It then returns the remainder of the string (after the opening tag) to allow it to be processed.

The `ChildNode` state, which seems quite complicated, turns out to require nothing but a simple conditional:

```python
class ChildNode: 
    def process(self, remaining_string, parser): 
        stripped = remaining_string.strip() 
        if stripped.startswith("</"): 
            parser.state = CloseTag() 
        elif stripped.startswith("<"): 
            parser.state = OpenTag() 
        else: 
            parser.state = TextNode() 
        return stripped 
```

The `strip()`  call removes whitespace from the string. Then the parser determines if  the next item is an opening or closing tag, or a string of text.  Depending on which possibility occurs, it sets the parser to a  particular state, and then tells it to parse the remainder of the  string.

The `OpenTag` state is similar to the `FirstTag` state, except that it adds the newly created node to the previous `current_node` object's `children` and sets it as the new `current_node`. It places the processor back in the `ChildNode` state before continuing:

```python
class OpenTag:
    def process(self, remaining_string, parser):
        i_start_tag = remaining_string.find("<")
        i_end_tag = remaining_string.find(">")
        tag_name = remaining_string[i_start_tag + 1 : i_end_tag]
        node = Node(tag_name, parser.current_node)
        parser.current_node.children.append(node)
        parser.current_node = node
        parser.state = ChildNode()
        return remaining_string[i_end_tag + 1 :]
```

The `CloseTag` state basically does the opposite; it sets the parser's `current_node` back to the parent node so any further children in the outside tag can be added to it:

```python
class CloseTag:
    def process(self, remaining_string, parser):
        i_start_tag = remaining_string.find("<")
        i_end_tag = remaining_string.find(">")
        assert remaining_string[i_start_tag + 1] == "/"
        tag_name = remaining_string[i_start_tag + 2 : i_end_tag]
        assert tag_name == parser.current_node.tag_name
        parser.current_node = parser.current_node.parent
        parser.state = ChildNode()
        return remaining_string[i_end_tag + 1 :].strip()
```

The two `assert` statements help ensure that the parse strings are consistent.

Finally, the `TextNode` state very simply extracts the text before the next close tag and sets it as a value on the current node:

```python
class TextNode: 
    def process(self, remaining_string, parser): 
        i_start_tag = remaining_string.find('<') 
        text = remaining_string[:i_start_tag] 
        parser.current_node.text = text 
        parser.state = ChildNode() 
        return remaining_string[i_start_tag:] 
```

Now we just have to set up the initial state on the parser object we created. The initial state is a `FirstTag` object, so just add the following to the `__init__` method:

```python
        self.state = FirstTag() 
```

To test the class, let's add a main script that opens an file from the command line, parses it, and prints the nodes:

```python
if __name__ == "__main__": 
    import sys 
    with open(sys.argv[1]) as file: 
        contents = file.read() 
        p = Parser(contents) 
        p.start() 
 
        nodes = [p.root] 
        while nodes: 
            node = nodes.pop(0) 
            print(node) 
            nodes = node.children + nodes 
```

This code opens the file, loads the contents, and parses the result. Then it prints each node and its children in order. The `__str__` method we originally added on the `node`  class takes care of formatting the nodes for printing. If we run the  script on the earlier example, it outputs the tree as follows:

```python
bookauthor: Dusty Phillipspublisher: Packt Publishingtitle: Python 3 Object Oriented Programmingcontentchapternumber: 1title: Object Oriented Designchapternumber: 2title: Objects In Python
```

Comparing this to the original simplified XML document tells us the parser is working.

# State Versus Strategy

The state pattern looks very similar to the strategy pattern; indeed, the UML diagrams  for the two are identical. The implementation, too, is identical. We  could even have written our states as first-class functions instead of  wrapping them in objects, as was suggested for strategy.

While the  two patterns have identical structures, they solve completely different  problems. The strategy pattern is used to choose an algorithm at  runtime; generally, only one of those algorithms is going to be chosen  for a particular use case. The state pattern, on the other hand, is  designed to allow switching between different states dynamically, as  some process evolves. In code, the primary difference is that the  strategy pattern is not typically aware of other strategy objects. In  the state pattern, either the state or the context needs to know which  other states that it can switch to.

# State Transition as Coroutines

The state pattern is the canonical object-oriented solution  to state-transition problems. However, you can get a similar effect by  constructing your objects as coroutines. Remember the regular expression  log file parser we built in **The Iterator Pattern**?  That was a state-transition problem in disguise. The main difference  between that implementation and one that defines all the objects (or  functions) used in the state pattern is that the coroutine solution  allows us to encode more of the boilerplate in language constructs.  There are two implementations, but neither one is inherently better than  the other. The state pattern is actually the only place I would  consider using coroutines outside of `asyncio`.