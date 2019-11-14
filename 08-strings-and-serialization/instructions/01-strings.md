Strings are a basic primitive  in Python; we've used them in nearly every example we've discussed so  far. All they do is represent an immutable sequence of characters.  However, though you may not have considered it before, *character* is  a bit of an ambiguous word; can Python strings represent sequences of  accented characters? Chinese characters? What about Greek, Cyrillic, or  Farsi?

In Python 3, the answer is yes. Python strings are all  represented in Unicode, a character definition standard that can  represent virtually any character in any language on the planet (and  some made-up languages and random characters as well). This is done  seamlessly. So, let's think of Python 3 strings as an immutable sequence  of Unicode characters. We've touched on many of the ways strings can be  manipulated in previous examples, but let's quickly cover it all in one  place: a crash course in string theory!

# String Manipulation

As you know, strings can be created  in Python by wrapping a sequence of characters in single or double  quotes. Multiline strings can easily be created using three quote  characters, and multiple hardcoded strings can be concatenated together  by placing them side by side. Here are some examples:

```python
a = "hello" 
b = 'world' 
c = '''a multiple 
line string''' 
d = """More 
multiple""" 
e = ("Three " "Strings " 
        "Together") 
```

That last string is  automatically composed into a single string by the interpreter. It is  also possible to concatenate strings using the `+` operator (as in `"hello " + "world"`).  Of course, strings don't have to be hardcoded. They can also come from  various outside sources, such as text files, user input, or can be  encoded on the network.

info> The  automatic concatenation of adjacent strings can make for some hilarious  bugs when a comma is missed. It is, however, extremely useful when a  long string needs to be placed inside a function call without exceeding  the 79 - character line-length limit suggested by the Python style  guide.

Like other sequences, strings can be iterated over (character by character), indexed, sliced, or concatenated. The syntax is the same as for lists.

The `str` class has numerous methods on it to make manipulating strings easier. The `dir` and `help` commands in the Python interpreter can tell us how to use all of them; we'll consider some of the more common ones directly.

Several  Boolean convenience methods help us identify whether or not the  characters in a string match a certain pattern. Here is a summary of  these methods. Most of these, such as `isalpha`, `isupper`/`islower`, and `startswith`/`endswith`, have obvious interpretations. The `isspace`  method is also fairly obvious, but remember that all whitespace  characters (including tab and newline) are considered, not just the  space character.

The `istitle` method returns `True`  if the first character of each word is capitalized and all other  characters are lowercase. Note that it does not strictly enforce the  English grammatical definition of title formatting. For example, Leigh  Hunt's poem **The Glove and the Lions** should be a valid title, even though not all words are capitalized. Robert Service's **The Cremation of Sam McGee** should also be a valid title, even though there is an uppercase letter in the middle of the last word.

Be careful with the `isdigit`, `isdecimal`, and `isnumeric`  methods, as they are more nuanced than we would expect. Many Unicode  characters are considered numbers besides the 10 digits we are used to.  Worse, the period character that we use to construct floats from strings  is not considered a decimal character, so `'45.2'.isdecimal()` returns `False`. The real decimal character is represented by Unicode value 0660, as in 45.2 (or `45\u06602`). Further, these methods do not verify whether the strings are valid numbers; `127.0.0.1` returns `True`  for all three methods. We might think we should use that decimal  character instead of a period for all numeric quantities, but passing  that character into the `float()` or `int()` constructor converts that decimal character to a zero:

```python
>>> float('45\u06602')4502.0
```

The  result of all these inconsistencies is that the Boolean numeric checks  are not very useful at all. We're usually much better off using a  regular expression (discussed later in this chapter) to confirm whether  the string matches a specific numeric pattern.

Other methods useful for pattern-matching do not return Booleans. The `count` method tells us how many times a given substring shows up in the string, while `find`, `index`, `rfind`, and `rindex` tell us the position of a given substring within the original string. The two `r` (for **right** or **reverse**) methods start searching from the end of the string. The `find` methods return `-1` if the substring can't be found, while `index` raises `ValueError` in this situation. Have a look at some of these methods in action:

```python
>>> s = "hello world">>> s.count('l')3>>> s.find('l')2>>> s.rindex('m')Traceback (most recent call last):  File "<stdin>", line 1, in <module>ValueError: substring not found
```

Most of the remaining string methods return transformations of the string. The `upper`, `lower`, `capitalize`, and `title` methods create new strings with all alphabetical characters in the given format. The `translate` method can use a dictionary to map arbitrary input characters to specified output characters.

For all of these methods, note that the input string remains unmodified; a brand new `str` instance is returned instead. If we need to manipulate the resultant string, we should assign it to a new variable, as in `new_value``=``value.capitalize()`.  Often, once we've performed the transformation, we don't need the old  value anymore, so a common idiom is to assign it to the same variable,  as in `value``=``value.title()`.

Finally, a couple of string methods return or operate on lists. The `split`  method accepts a substring and splits the string into a list of strings  wherever that substring occurs. You can pass a number as a second  parameter to limit the number of resultant strings. The `rsplit`method behaves identically to `split` if you don't limit the number of strings, but if you do supply a limit, it starts splitting from the end of the string. The `partition` and `rpartition `methods  split the string at only the first or last occurrence of the substring,  and return a tuple of three values: characters before the substring,  the substring itself, and the characters after the substring.

As the inverse of `split`, the `join`  method accepts a list of strings, and returns all of those strings  combined together by placing the original string between them. The `replace` method accepts two arguments, and returns a string where each instance of the first argument has been replaced with the second. Here are some of these methods in action:

```python
>>> s = "hello world, how are you">>> s2 = s.split(' ')>>> s2['hello', 'world,', 'how', 'are', 'you']>>> '#'.join(s2)'hello#world,#how#are#you'>>> s.replace(' ', '**')'hello**world,**how**are**you'>>> s.partition(' ')('hello', ' ', 'world, how are you')
```

There you have it, a whirlwind tour of the most common methods on the `str` class! Now, let's look at Python 3's method for composing strings and variables to create new strings.

# String Formatting

Python 3 has powerful string formatting  and templating mechanisms that allow us to construct strings comprised  of hardcoded text and interspersed variables. We've used it in many  previous examples, but it is much more versatile than the simple  formatting specifiers we've used.

A string can be turned into a format string (also called an **f-string**) by prefixing the opening quotation mark with an f, as in `f"hello world"`. If such a string contains the special characters  `{` and `}`, variables from the surrounding scope can be used to replace them as in this example:

```python
name = "Dusty"
activity = "writing"
formatted = f"Hello {name}, you are currently {activity}."
print(formatted)
```

If we run these statements, it replaces the braces with variables, in order:

```python
Hello Dusty, you are currently writing.
```

## Escaping Braces

Brace characters are often useful  in strings, aside from formatting. We need a way to escape them in  situations where we want them to be displayed as themselves, rather than  being replaced. This can be done by doubling the braces. For example,  we can use Python to format a basic Java program:

```python
classname = "MyClass"
python_code = "print('hello world')"
template = f"""
public class {classname} {{
    public static void main(String[] args) {{
        System.out.println("{python_code}");
    }}
}}"""

print(template)
```

Where we see the `{{` or `}}`  sequence in the template—that is, the braces enclosing the Java class  and method definition—we know the f-string will replace them with single  braces, rather than some argument in the surrounding methods. Here's  the output:

```python
public class MyClass {    public static void main(String[] args) {        System.out.println("print('hello world')");    }}
```

The class name and contents of the output  have been replaced with two parameters, while the double braces have  been replaced with single braces, giving us a valid Java file. Turns  out, this is about the simplest possible Python program to print the  simplest possible Java program that can print the simplest possible  Python program.

# f-strings can Contain Python Code

We aren't restricted to passing simple string variables  into an f-string method. Any primitives, such as integers or floats,  can be formatted. More interestingly, complex objects, including lists,  tuples, dictionaries, and arbitrary objects can be used, and we can  access indexes and variables or call functions on those objects from  within the `format` string.

For example, if our email message had grouped the `From` and `To`  email addresses into a tuple, and placed the subject and message in a  dictionary, for some reason (perhaps because that's the input required  for an existing `send_mail` function we want to use), we can format it like this:

```python
emails = ("a@example.com", "b@example.com")
message = {
    "subject": "You Have Mail!",
    "message": "Here's some mail for you!",
}

formatted = f"""
From: <{emails[0]}>
To: <{emails[1]}>
Subject: {message['subject']}
{message['message']}"""
print(formatted)
```

The variables inside the braces in  the template string look a little weird, so let's look at what they're  doing. The two email addresses are looked up by `emails[x]`, where `x` is either `0` or `1`.  The square brackets with a number inside are the same kind of index lookup we see in regular Python code, so `emails[0]`  refers to the first item in the `emails` tuple. The indexing syntax works with any indexable object, so we see similar behavior when we access `message[subject]`, except this time  we are looking up a string key in a dictionary. Notice that, unlike in  Python code, we do not need to put quotes around the string in the  dictionary lookup.

We can even do multiple levels of lookup if we have nested data structures. If we modify the above code to put the `emails` tuple inside the `message` dictionary, we can use an indexed lookup as follows:

```python
message["emails"] = emails

formatted = f"""
From: <{message['emails'][0]}>
To: <{message['emails'][1]}>
Subject: {message['subject']}
{message['message']}"""
print(formatted)
```

I would recommend against doing this often, as template strings rapidly become difficult to understand.

Alternatively,  if you have an object or class, you can execute object lookups or even  call methods inside the f-string. Let's change our email message data once again, this time to a class:

```python
class EMail:
    def __init__(self, from_addr, to_addr, subject, message):
        self.from_addr = from_addr
        self.to_addr = to_addr
        self.subject = subject
        self._message = message

    def message(self):
        return self._message


email = EMail(
    "a@example.com",
    "b@example.com",
    "You Have Mail!",
    "Here's some mail for you!",
)

formatted = f"""
From: <{email.from_addr}>
To: <{email.to_addr}>
Subject: {email.subject}

{email.message()}"""
print(formatted)
```

The template in this example may be more readable than the previous examples, but the overhead of creating an `email`  class adds complexity to the Python code. It would be foolish to create  a class for the express purpose of including the object in a template.  Typically, we'd use this sort of lookup if the object we are trying to  format already exists.

Pretty much any Python code that you would expect to return a string (or a value that can convert to a string with the `str()`  function) can be executed inside an f-string.  As an example of how  powerful it can get, you can even use a list comprehension or ternary  operator in a format string parameter:

```python
>>> f"['a' for a in range(5)]"
"['a' for a in range(5)]"
>>> f"{'yes' if True else 'no'}"
'yes'
```

## Making it Look Right

It's nice to be able to include variables  in template strings, but sometimes the variables need a bit of coercion  to make them look the way we want them to in the output. For example,  if we are performing calculations with currency, we may end up with a  long decimal that we don't want to show up in our template:

```python
subtotal = 12.32
tax = subtotal * 0.07
total = subtotal + tax

print(
    "Sub: ${0} Tax: ${1} Total: ${total}".format(
        subtotal, tax, total=total
    )
)
```

If we run this formatting code, the output doesn't quite look like proper currency:

```python
Sub: $12.32 Tax: $0.8624 Total: $13.182400000000001
```

info> Technically, we should never use floating-point numbers in currency calculations like this; we should construct `decimal.Decimal()`  objects instead. Floats are dangerous because their calculations are  inherently inaccurate beyond a specific level of precision. But we're  looking at strings, not floats, and currency is a great example for  formatting!

To fix the preceding `format`  string, we can include some additional information inside the curly  braces to adjust the formatting of the parameters. There are tons of  things we can customize, but the basic syntax inside the braces is the  same. After providing the template value, we include a colon, and then some specific syntax for the formatting. Here's an improved version:

```python
print(
    "Sub: ${0:0.2f} Tax: ${1:0.2f} "
    "Total: ${total:0.2f}".format(subtotal, tax, total=total)
)
```

The `0.2f` format specifier after the colons basically says the following, from left to right:

- `0`: for values lower than one, make sure a zero is displayed on the left-hand of the decimal point
- `.`: show a decimal point
- `2`: show two places after the decimal
- `f`: format the input value as a float

We  can also specify that each number should take up a particular number of  characters on the screen by placing a value before the period. This can  be useful for outputting tabular data, for example:

```python
orders = [("burger", 2, 5), ("fries", 3.5, 1), ("cola", 1.75, 3)]

print("PRODUCT QUANTITY PRICE SUBTOTAL")
for product, price, quantity in orders:
    subtotal = price * quantity
    print(
        f"{product:10s}{quantity: ^9d} "
        f"${price: <8.2f}${subtotal: >7.2f}"
    )
```

OK, that's a pretty scary-looking format string, so let's see how it works before we break it down into understandable parts:

```python
PRODUCT    QUANTITY    PRICE    SUBTOTALburger        5        $2.00    $  10.00fries         1        $3.50    $   3.50cola          3        $1.75    $   5.25
```

Nifty! So, how is this actually happening? We have four variables we are formatting, in each line of the `for` loop. The first variable is a string that is formatted with `{product:10s}`. This one is easier to read from right to left:

-  `s` means it is a string variable.
- `10`  means it should take up 10 characters. By default, with strings, if the  string is shorter than the specified number of characters, it appends  spaces to the right-hand side of the string to make it long enough  (beware, however: if the original string is too long, it won't be  truncated!).
- `product:`, of course, is the name of the variable or Python expression being formatted.

The formatter for the `quantity` value is`{quantity: ^9d}`. You can interpret this format from right to left as follows:

- `d` represents an integer value.
- `9` tells us the value should take up nine characters on the screen.
- `^`  tells us that the number should be aligned in the center of this  available padding; this makes the column look a bit more professional.
- (space)  tells the formatter to use a space as the padding character. With  integers, instead of spaces, the extra characters are zeros, by default.
- `quantity:` is the variable being formatted.

All  these specifiers have to be in the right order, although all are  optional: fill first, then align, then the size, and finally, the type.

We do similar things with the specifiers for `price` and `subtotal`. For `price`, we use `{2:``<8.2f}`; and for `subtotal`, `{3:``>7.2f}`. In both cases, we're specifying a space as the fill character, but we use the `<` and `>`  symbols, respectively, to represent that the numbers should be aligned  to the left or right within a minimum space of eight or seven  characters. Further, each float should be formatted to two decimal  places.

The **type** character for different types can affect formatting output as well. We've seen the `s`, `d`, and `f` types, for strings, integers, and floats. Most of the other format specifiers are alternative versions of these; for example, `o` represents octal format and `X` represents hexadecimal if formatting integers. The `n` type specifier can be useful for formatting integer separators in the current locale's format. For floating-point numbers, the `%` type will multiply by 100 and format a float as a percentage.

## Custom Formatters

While these standard formatters apply to most built-in objects, it is also possible for other objects to define nonstandard specifiers. For example, if we pass a `datetime` object into `format`, we can use the specifiers used in the `datetime.strftime` function, as follows:

```python
import datetime 
print("{the_date:%Y-%m-%d %I:%M%p }".format( 
    datetime.datetime.now())) 
```

It is even possible  to write custom formatters for objects we create ourselves, but that is  beyond the scope of this book. Look into overriding the `__format__`special method if you need to do this in your code.

The  Python formatting syntax is quite flexible but it is a difficult  mini-language to remember. I use it every day and still occasionally  have to look up forgotten concepts in the documentation. It also isn't  powerful enough for serious templating needs, such as generating web  pages. There are several third-party templating libraries you can look  into if you need to do more than basic formatting of a few strings.

## The Format Method

There  are a few cases where you won't be able to use f-strings. First, you  can't reuse a single template string with different variables. Second,  f-strings were introduced in Python 3.6. If you're stuck on an older version of Python or need to reuse template strings, you can use the older `str.format` method instead. It uses the same formatting specifiers as f-strings, but can be called multiple times on one string. Here's an example:

```python
>>> template = "abc {number:*^10d}"
>>> template.format(number=32)
'abc ****32****'
>>> template.format(number=84)
'abc ****84****'
```

The `format` method behaves similarly to f-strings, but there are a couple of differences:

- It  is restricted in what it can look up. You can access attributes on  objects or look up an index in a list or dict, but you can't call a  function inside the template string.
- You can use integers to access positional arguments passed to the format method: `"{0} world".format('bonjour')`. The indexes are optional if you specify the variables in order: `"{} {}".format('hello', 'world')`.

# Strings are Unicode

At the beginning of this section, we defined strings as collections of immutable  Unicode characters. This actually makes things very complicated at  times, because Unicode isn't really a storage format. If you get a  string of bytes from a file or a socket, for example, they won't be in  Unicode. They will, in fact, be the built-in type `bytes`.  Bytes are immutable sequences of... well, bytes. Bytes are the basic  storage format in computing. They represent 8 bits, usually described as  an integer between 0 and 255, or a hexadecimal equivalent between 0 and  FF. Bytes don't represent anything specific; a sequence of bytes may  store characters of an encoded string, or pixels in an image.

If  we print a byte object, any bytes that map to ASCII representations will  be printed as their original character, while non-ASCII bytes (whether  they are binary data or other characters) are printed as hex codes  escaped by the `\x` escape sequence. You may  find it odd that a byte, represented as an integer, can map to an ASCII  character. But ASCII is really just code where each letter is  represented by a different byte pattern, and therefore, a different  integer. The character *a* is  represented by the same byte as the integer 97, which is the hexadecimal  number 0x61. Specifically, all of these are an interpretation of the  binary pattern 01100001.

Many I/O operations only know how to deal with `bytes`, even if the `bytes` object refers to textual data. It is therefore vital to know how to convert between `bytes` and Unicode.

The problem is that there are many ways to map `bytes`  to Unicode text. Bytes are machine-readable values, while text is a  human-readable format. Sitting in between is an encoding that maps a  given sequence of bytes to a given sequence of text characters.

However,  there are multiple such encodings (ASCII is only one of them). The same  sequence of bytes represents completely different text characters when  mapped using different encodings! So, `bytes`  must be decoded using the same character set with which they were  encoded. It's not possible to get text from bytes without knowing how  the bytes should be decoded. If we receive unknown bytes without a specified encoding, the best we can do is guess what format they are encoded in, and we may be wrong.

## Converting bytes to text

If we have an array of `bytes` from somewhere, we can convert it to Unicode using the `.decode` method on the `bytes` class. This method accepts a string for the name of the character encoding. There are many such names; common ones for Western languages include ASCII, UTF-8, and latin-1.

The  sequence of bytes (in hex), 63 6c 69 63 68 e9, actually represents the  characters of the word cliché in latin-1 encoding. The following example  will encode this sequence of bytes and convert it to a Unicode string  using latin-1 encoding:

```python
characters = b'\x63\x6c\x69\x63\x68\xe9' 
print(characters) 
print(characters.decode("latin-1")) 
```

The first line creates a `bytes` object. Analogous to an f-string, the `b` character immediately before the string tells us that we are defining a `bytes`  object instead of a normal Unicode string. Within the string, each byte  is specified using—in this case—a hexadecimal number. The `\x` character escapes within the byte string, and each say, **the next two characters represent a byte using hexadecimal digits**.

Provided we are using a shell that understands latin-1 encoding, the two `print` calls will output the following strings:

```python
b'clich\xe9'cliché
```

The first `print`  statement renders the bytes for ASCII characters as themselves. The  unknown (unknown to ASCII, that is) character stays in its escaped hex  format. The output includes a `b` character at the beginning of the line to remind us that it is a `bytes` representation, not a string.

The next call decodes the string using latin-1 encoding. The `decode` method returns a normal (Unicode) string with the correct characters. However, if we had decoded this same string using the Cyrillic `iso8859-5` encoding, we'd have ended up with the `'clichщ'`string! This is because the `\xe9` byte maps to different characters in the two encodings.

## Converting Text to Bytes

If we need to convert incoming bytes into Unicode, we're clearly also going to have situations where we convert outgoing Unicode into byte sequences. This is done with the `encode` method on the `str` class, which, like the `decode` method, requires a character set. The following code creates a Unicode string and encodes it in different character sets:

```python
characters = "cliché" 
print(characters.encode("UTF-8")) 
print(characters.encode("latin-1")) 
print(characters.encode("CP437")) 
print(characters.encode("ascii")) 
```

The first three  encodings create a different set of bytes for the accented character.  The fourth one can't even handle that byte:

```python
b'clich\xc3\xa9'b'clich\xe9'b'clich\x82'Traceback (most recent call last):  File "1261_10_16_decode_unicode.py", line 5, in <module>    print(characters.encode("ascii"))UnicodeEncodeError: 'ascii' codec can't encode character '\xe9' in position 5: ordinal not in range(128)
```

Now  you should understand the importance of encodings! The accented  character is represented as a different byte for each encoding; if we  use the wrong one when we are decoding bytes to text, we get the wrong  character.

The exception in the last case is not always the  desired behavior; there may be cases where we want the unknown  characters to be handled in a different way. The `encode` method takes an optional string argument named `errors` that can define how such characters should be handled. This string can be one of the following:

- `strict`
- `replace`
- `ignore`
- `xmlcharrefreplace`

The `strict`  replacement strategy is the default we just saw. When a byte sequence  is encountered that does not have a valid representation in the  requested encoding, an exception is raised. When the `replace`  strategy is used, the character is replaced with a different character;  in ASCII, it is a question mark; other encodings may use different  symbols, such as an empty box. The `ignore` strategy simply discards any bytes it doesn't understand, while the `xmlcharrefreplace` strategy creates an `xml`  entity representing the Unicode character. This can be useful when  converting unknown strings for use in an XML document. Here's how each  of the strategies affects our sample word:

| **Strategy**        | **Result of applying**`"cliché".encode("ascii", strategy)` |
| ------------------- | ---------------------------------------------------------- |
| `replace`           | `b'clich?'`                                                |
| `ignore`            | `b'clich'`                                                 |
| `xmlcharrefreplace` | `b'cliché'`                                                |

It is possible to call the `str.encode` and `bytes.decode`  methods without passing an encoding name. The encoding will be set to  the default encoding for the current platform. This will depend on the  current operating system and locale or regional settings; you can look  it up using the `sys.getdefaultencoding()`  function. It is usually a good idea to specify the encoding explicitly,  though, since the default encoding for a platform may change, or the  program may one day be extended to work on text from a wider variety of  sources.

If you are encoding text and don't know which encoding to  use, it is best to use UTF-8 encoding. UTF-8 is able to represent any  Unicode character. In modern software, it is a de facto standard  encoding to ensure documents in any language—or even multiple languages  —can be exchanged. The various other possible encodings are useful for  legacy documents or in regions that still use different character sets by default.

The  UTF-8 encoding uses one byte to represent ASCII and other common  characters, and up to four bytes for more complex characters. UTF-8 is  special because it is backwards-compatible with ASCII; any ASCII  document encoded using UTF-8 will be identical to the original ASCII  document.

info> I can never remember whether to use `encode` or `decode` to convert from binary bytes to Unicode. I always wished these methods were named `to_binary` and `from_binary` instead. If you have the same problem, try mentally replacing the word *code* with *binary*; *enbinary* and *debinary* are pretty close to *to_binary* and *from_binary*. I have saved a lot of time by not looking up the method help files since devising this mnemonic.

# Mutable Byte Strings

The `bytes` type, like `str`, is immutable. We can use index and slice notation on a `bytes`  object and search for a particular sequence of bytes, but we can't  extend or modify them. This can be very inconvenient when dealing with  I/O, as it is often necessary to buffer incoming or outgoing bytes until  they are ready to be sent. For example, if we are receiving data from a  socket, it may take several `recv` calls before we have received an entire message.

This is where the `bytearray`  built-in comes in. This type behaves something like a list, except it  only holds bytes. The constructor for the class can accept a `bytes` object to initialize it. The `extend` method can be used to append another `bytes` object to the existing array (for example, when more data comes from a socket or other I/O channel).

Slice notation can be used on `bytearray` to modify the item inline. For example, this code constructs a `bytearray` from a `bytes` object and then replaces two bytes:

```python
b = bytearray(b"abcdefgh") 
b[4:6] = b"\x15\xa3" 
print(b) 
```

The output looks like this:

```python
bytearray(b'abcd\x15\xa3gh')
```

If we want to manipulate a single element in `bytearray`, we must pass an integer between 0 and 255 (inclusive) as the value. This integer represents a specific `bytes` pattern. If we try to pass a character or `bytes` object, it will raise an exception.

A single byte character can be converted to an integer using the `ord` (short for ordinal) function. This function returns the integer representation of a single character:

```python
b = bytearray(b"abcdef")
b[3] = ord(b"g")
b[4] = 68
print(b)
```

The output looks like this:

```python
bytearray(b'abcgDf')
```

After constructing the array, we replace the character at index `3` (the fourth character, as indexing starts at `0`, as with lists) with byte `103`. This integer was returned by the `ord` function and is the ASCII character for the lowercase `g`. For illustration, we also replaced the next character up with byte number `68`, which maps to the ASCII character for the uppercase `D`.

The `bytearray` type has methods that allow it to behave like a list (we can append integer bytes to it, for example), but also like a `bytes` object; we can use methods such as `count` and `find` the same way they would behave on a `bytes` or `str` object. The difference is that `bytearray` is a mutable type, which can be useful for building up complex sequences of bytes from a specific input source.