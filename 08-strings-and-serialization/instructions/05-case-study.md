Let's build a basic regular expression-powered templating engine in  Python. This engine will parse a text file (such as an HTML page) and  replace certain directives  with text calculated from the input to those directives. This is about  the most complicated task we would want to do with regular expressions;  indeed, a full-fledged version of this would likely utilize a proper  language-parsing mechanism.

Consider the following input file:

```html
/** include header.html **/ 
<h1>This is the title of the front page</h1> 
/** include menu.html **/ 
<p>My name is /** variable name **/. 
This is the content of my front page. It goes below the menu.</p> 
<table> 
<tr><th>Favourite Books</th></tr> 
/** loopover book_list **/ 
<tr><td>/** loopvar **/</td></tr> 
 
/** endloop **/ 
</table> 
/** include footer.html **/ 
Copyright &copy; Today 
```

This file contains **tags** of the form `/** <directive> <data> **/` where the data is an optional single word and the directives are as follows:

- `include`: Copy the contents of another file here
- `variable`: Insert the contents of a variable here
- `loopover`: Repeat the contents of the loop for a variable that is a list
- `endloop`: Signal the end of looped text
- `loopvar`: Insert a single value from the list being looped over

This template will render a different page depending which variables are passed into it. These variables will be passed in from a so-called context file. This will be encoded as a `json` object with keys representing the variables in question. My context file might look like this, but you would derive your own:

```python
{ 
    "name": "Dusty", 
    "book_list": [ 
        "Thief Of Time", 
        "The Thief", 
        "Snow Crash", 
        "Lathe Of Heaven" 
    ] 
} 
```

Before we get into the actual string processing,  let's throw together some object-oriented boilerplate code for  processing files and grabbing data from the command line:

```python
import re 
import sys 
import json 
from pathlib import Path 
 
DIRECTIVE_RE = re.compile( 
    r'/\*\*\s*(include|variable|loopover|endloop|loopvar)' 
    r'\s*([^ *]*)\s*\*\*/') 
 
 
class TemplateEngine: 
    def __init__(self, infilename, outfilename, contextfilename): 
        self.template = open(infilename).read() 
        self.working_dir = Path(infilename).absolute().parent 
        self.pos = 0 
        self.outfile = open(outfilename, 'w') 
        with open(contextfilename) as contextfile: 
            self.context = json.load(contextfile) 
 
    def process(self): 
        print("PROCESSING...") 
 
 
if __name__ == '__main__': 
    infilename, outfilename, contextfilename = sys.argv[1:] 
    engine = TemplateEngine(infilename, outfilename, contextfilename) 
    engine.process() 
```

This is all pretty basic, we create a class and initialize it with some variables passed in on the command line.

Notice  how we try to make the regular expression a little bit more readable by  breaking it across two lines? We use raw strings (the r prefix), so we  don't have to double escape all our backslashes. This is common in  regular expressions, but it's still a mess. (Regular expressions always  are, but they're often worth it.)

The `pos` indicates the current character in the content that we are processing; we'll see a lot more of it in a moment.

Now all that's left is to implement the process method. There are a few ways to do this. Let's do it in a fairly explicit way.

The  process method has to find each directive that matches the regular  expression and do the appropriate work with it. However, it also has to  take care of outputting the normal text before, after, and between each  directive to the output file, unmodified.

One good feature of the compiled version of regular expressions is that we can tell the `search` method to start searching at a specific position by passing the `pos` keyword argument. If we temporarily define doing the appropriate work with a directive as **ignore the directive and delete it from the output file**, our process loop looks quite simple:

```python
def process(self): 
    match = DIRECTIVE_RE.search(self.template, pos=self.pos) 
    while match: 
        self.outfile.write(self.template[self.pos:match.start()]) 
        self.pos = match.end() 
        match = DIRECTIVE_RE.search(self.template, pos=self.pos) 
    self.outfile.write(self.template[self.pos:]) 
```

In English, this function finds the first string  in the text that matches the regular expression, outputs everything  from the current position to the start of that match, and then advances  the position to the end of the aforesaid match. Once it's out of  matches, it outputs everything since the last position.

Of course, ignoring the directive is pretty useless in a templating engine, so let's replace that position advancing line with code that delegates to a different method on the class depending on the directive:

```python
def process(self): 
    match = DIRECTIVE_RE.search(self.template, pos=self.pos) 
    while match: 
        self.outfile.write(self.template[self.pos:match.start()]) 
        directive, argument = match.groups() 
        method_name = 'process_{}'.format(directive) 
        getattr(self, method_name)(match, argument) 
        match = DIRECTIVE_RE.search(self.template, pos=self.pos) 
    self.outfile.write(self.template[self.pos:]) 
```

So,  we grab the directive and the single argument from the regular  expression. The directive becomes a method name and we dynamically look  up that method name on the `self` object (a  little error processing here, in case the template writer provides an  invalid directive, would be better). We pass the `match` object and argument into that method and assume that method will deal with everything appropriately, including moving the `pos` pointer.

Now  that we've got our object-oriented architecture this far, it's actually  pretty simple to implement the methods that are delegated to. The `include` and `variable` directives are totally straightforward:

```python
def process_include(self, match, argument): 
    with (self.working_dir / argument).open() as includefile: 
        self.outfile.write(includefile.read()) 
        self.pos = match.end() 
 
def process_variable(self, match, argument): 
    self.outfile.write(self.context.get(argument, '')) 
    self.pos = match.end() 
```

The first simply looks up the included  file and inserts the file contents, while the second looks up the  variable name in the context dictionary (which was loaded from `json` in the `__init__` method), defaulting to an empty string if it doesn't exist.

The  three methods that deal with looping are a bit more intense, as they  have to share state between the three of them. For simplicity (I'm sure  you're eager to see the end of this long chapter—we're almost there!),  we'll handle this case using instance variables on the  class itself. As an activity, you might want to consider better ways to  architect this, especially after reading the next three lessons:

```python
    def process_loopover(self, match, argument): 
        self.loop_index = 0 
        self.loop_list = self.context.get(argument, []) 
        self.pos = self.loop_pos = match.end() 
 
    def process_loopvar(self, match, argument): 
        self.outfile.write(self.loop_list[self.loop_index]) 
        self.pos = match.end() 
 
    def process_endloop(self, match, argument): 
        self.loop_index += 1 
        if self.loop_index >= len(self.loop_list): 
            self.pos = match.end() 
            del self.loop_index 
            del self.loop_list 
            del self.loop_pos 
        else: 
            self.pos = self.loop_pos 
```

When we encounter the `loopover` directive, we don't have to output anything, but we do have to set the initial state on three variables. The `loop_list` variable is assumed to be a list pulled from the context dictionary. The `loop_index` variable indicates which position in that list should be output in this iteration of the loop, while `loop_pos` is stored so we know where to jump back to when we get to the end of the loop.

The `loopvar` directive outputs the value at the current position in the `loop_list` variable and skips to the end of the directive. Note that it doesn't increment the loop index, because the `loopvar` directive could be called multiple times inside a loop.

The `endloop` directive is more complicated. It determines whether there are more elements in the `loop_list`;  if there are, it just jumps back to the start of the loop, incrementing  the index. Otherwise, it resets all the variables that were being used  to process the loop and jumps to the end of the directive so the engine  can carry on with the next match.

Note that this particular looping mechanism is very fragile; if a template designer were to try nesting loops or to forget an `endloop`  call, it would go poorly for them. We would need a lot more error  checking and would probably want to store more loop state to make this a  production platform. But I promised that the end of the chapter was  nigh, so let's just head to the exercises, after seeing how our sample  template is rendered with its context:

```html
<html>

<body>

<h1>This is the title of the front page</h1>
<a href="link1.html">First Link</a>
<a href="link2.html">Second Link</a>

<p>My name is Dusty. This is the content of my front page. It goes below the menu.</p>
<table>
    <tr>
        <th>Favourite Books</th>
    </tr>

    <tr>
        <td>Thief Of Time</td>
    </tr>


    <tr>
        <td>The Thief</td>
    </tr>


    <tr>
        <td>Snow Crash</td>
    </tr>


    <tr>
        <td>Lathe Of Heaven</td>
    </tr>


</table>
</body>

</html>
 Copyright &copy; Today
```

There are some weird newline effects due to the way we planned our template, but it works as expected.