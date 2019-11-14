You know what's really hard to do using  object-oriented principles? Parsing strings to match arbitrary  patterns, that's what. There have been a fair number of academic papers  written in which object-oriented design is used to set up  string-parsing, but the result is always very verbose and hard to read,  and they are not widely used in practice.

In the real world,  string-parsing in most programming languages is handled by regular  expressions. These are not verbose, but, wow, are they ever hard to  read, at least until you learn the syntax. Even though regular  expressions are not object-oriented, the Python regular expression  library provides a few classes and objects that you can use to construct  and run regular expressions.

Regular expressions are used to  solve a common problem: Given a string, determine whether that string  matches a given pattern and, optionally, collect substrings that contain  relevant information. They can be used to answer questions such as the  following:

- Is this string a valid URL?
- What is the date and time of all warning messages in a log file?
- Which users in `/etc/passwd` are in a given group?
- What username and document were requested by the URL a visitor typed?

There are many similar scenarios where  regular expressions are the correct answer. Many programmers have made  the mistake of implementing complicated and fragile string-parsing  libraries because they didn't know or wouldn't learn regular  expressions. In this section, we'll gain enough knowledge of regular  expressions to not make such mistakes.

# Matching Patterns

Regular expressions are a complicated  mini-language. They rely on special characters to match unknown  strings, but let's start with literal characters, such as letters,  numbers, and the space character, which always match themselves. Let's  see a basic example:

```python
import re 
 
search_string = "hello world" 
pattern = "hello world" 
 
match = re.match(pattern, search_string) 
 
if match: 
    print("regex matches") 
```

The Python Standard Library module for regular expressions is called `re`.  We import it and set up a search string and pattern to search for; in  this case, they are the same string. Since the search string matches the  given pattern, the conditional passes and the `print` statement executes.

Bear in mind that the `match` function matches the pattern to the beginning of the string. Thus, if the pattern were `"ello world"`, no match would be found. With confusing asymmetry, the parser stops searching as soon as it finds a match, so the pattern `"hello wo"`  matches successfully. Let's build a small example program to  demonstrate these differences and help us learn other regular expression  syntax:

```python
import sys 
import re 
 
pattern = sys.argv[1] 
search_string = sys.argv[2] 
match = re.match(pattern, search_string) 
 
if match: 
    template = "'{}' matches pattern '{}'" 
else: 
    template = "'{}' does not match pattern '{}'" 
 
print(template.format(search_string, pattern)) 
```

This  is just a generic version of the earlier example that accepts the  pattern and search string from the command line. We can see how the  start of the pattern must match, but a value is returned as soon as a match is found in the following command-line interaction:

```bash
$ python regex_generic.py "hello worl" "hello world"'hello world' matches pattern 'hello worl'$ python regex_generic.py "ello world" "hello world"'hello world' does not match pattern 'ello world'
```

We'll be using this script throughout the next few sections. While the script is always invoked with the `python regex_generic.py "<pattern>" "<string>"` command, we'll only see the output in the following examples, to conserve space.

If  you need control over whether items happen at the beginning or end of a  line (or if there are no newlines in the string, or at the beginning  and end of the string), you can use the `^` and `$`  characters to represent the start and end of the string respectively.  If you want a pattern to match an entire string, it's a good idea to  include both of these:

```python
'hello world' matches pattern '^hello world$''hello worl' does not match pattern '^hello world$'
```

## Matching a Selection of Characters

Let's start with matching an arbitrary  character. The period character, when used in a regular expression  pattern, can match any single character. Using a period in the string  means you don't care what the character is, just that there is a  character there. Here are some examples:

```python
'hello world' matches pattern 'hel.o world''helpo world' matches pattern 'hel.o world''hel o world' matches pattern 'hel.o world''helo world' does not match pattern 'hel.o world'
```

Notice how the last example does not match because there is no character at the period's position in the pattern.

That's  all well and good, but what if we only want a few specific characters  to match? We can put a set of characters inside square brackets to match  any one of those characters. So, if we encounter the string `[abc]`   in a regular expression pattern, we know that those five (including the  two square brackets) characters will only match one character in the  string being searched, and further, that this one character will be  either an `a`, a `b`, or a `c`. Let's see a few examples:

```python
'hello world' matches pattern 'hel[lp]o world''helpo world' matches pattern 'hel[lp]o world''helPo world' does not match pattern 'hel[lp]o world'
```

These square bracket sets should be named character sets, but they are more often referred to as **character classes**.  Often, we want to include a large range of characters inside these  sets, and typing them all out can be monotonous and error-prone.  Fortunately, the regular expression designers thought of this and gave  us a shortcut. The dash character, in a character set, will create a range. This is especially useful if you want to match **all lowercase letters**, **all letters**, or **all numbers,** as follows:

```python
 'hello   world' does not match pattern 'hello [a-z] world'
 'hello b world' matches pattern 'hello [a-z] world''hello B world' matches pattern 'hello [a-zA-Z] world''hello 2 world' matches pattern 'hello [a-zA-Z0-9] world'
```

There  are other ways to match or exclude individual characters, but you'll  need to find a more comprehensive tutorial via a web search if you want  to find out what they are!

## Escaping Characters

If putting a period character in a pattern  matches any arbitrary character, how do we match just a period in a  string? One way might be to put the period inside square brackets to  make a character class, but a more generic method is to use backslashes  to escape it. Here's a regular expression to match two-digit decimal  numbers between 0.00 and 0.99:

```python
'0.05' matches pattern '0\.[0-9][0-9]''005' does not match pattern '0\.[0-9][0-9]''0,05' does not match pattern '0\.[0-9][0-9]'
```

For this pattern, the two characters `\.` match the single `.` character. If the period character is missing or is a different character, it will not match.

This backslash escape sequence is used for a variety of special characters in regular expressions. You can use `\[` to insert a square bracket without starting a character class, and `\(` to insert a parenthesis, which we'll later see is also a special character.

More interestingly, we can also use the escape symbol followed by a character to represent special characters such as newlines (`\n`) and tabs (`\t`). Further, some character classes can be represented more succinctly using escape strings: `\s` represents whitespace characters; `\w` represents letters, numbers, and underscore; and `\d` represents a digit:

```python
'(abc]' matches pattern '\(abc\]'' 1a' matches pattern '\s\d\w''\t5n' does not match pattern '\s\d\w''5n' matches pattern '\s\d\w'
```

## Matching Multiple Characters

With this information, we can match most  strings of a known length, but most of the time, we don't know how many  characters to match inside a pattern. Regular expressions can take care  of this, too. We can modify a pattern by appending one of several  hard-to-remember punctuation symbols to match multiple characters.

The asterisk (`*`)  character says that the previous pattern can be matched zero or more  times. This probably sounds silly, but it's one of the most useful  repetition characters. Before we explore why, consider some silly  examples to make sure we understand what it does:

```python
'hello' matches pattern 'hel*o''heo' matches pattern 'hel*o''helllllo' matches pattern 'hel*o'
```

So, the `*` character in the pattern says that the previous pattern (the `l`  character) is optional, and if present, can be repeated as many times  as possible to match the pattern. The rest of the characters (`h`, `e`, and `o`) have to appear exactly once.

It's  pretty rare to want to match a single letter multiple times, but it  gets more interesting if we combine the asterisk with patterns that  match multiple characters. So, `.*`, for example, will match any string, whereas `[a-z]*` matches any collection of lowercase words, including the empty string. Here are a few examples:

```python
'A string.' matches pattern '[A-Z][a-z]* [a-z]*\.''No .' matches pattern '[A-Z][a-z]* [a-z]*\.''' matches pattern '[a-z]*.*'
```

The plus (`+`) sign in a pattern behaves similarly  to an asterisk; it states that the previous pattern can be repeated one  or more times, but, unlike the asterisk, is not optional. The question  mark (`?`) ensures a pattern shows up exactly  zero or one times, but not more. Let's explore some of these by playing  with numbers (remember that `\d` matches the same character class as `[0-9]`:

```python
'0.4' matches pattern '\d+\.\d+''1.002' matches pattern '\d+\.\d+''1.' does not match pattern '\d+\.\d+''1%' matches pattern '\d?\d%''99%' matches pattern '\d?\d%''999%' does not match pattern '\d?\d%'
```

## Grouping Patterns Together

So far, we've seen how we can repeat  a pattern multiple times, but we are restricted in what patterns we can  repeat. If we want to repeat individual characters, we're covered, but  what if we want a repeating sequence of characters? Enclosing any set of  patterns in parentheses allows them to be treated as a single pattern  when applying repetition operations. Compare these patterns:

```python
'abccc' matches pattern 'abc{3}''abccc' does not match pattern '(abc){3}''abcabcabc' matches pattern '(abc){3}'
```

Combined  with complex patterns, this grouping feature greatly expands our  pattern-matching repertoire. Here's a regular expression that matches  simple English sentences:

```python
'Eat.' matches pattern '[A-Z][a-z]*( [a-z]+)*\.$''Eat more good food.' matches pattern '[A-Z][a-z]*( [a-z]+)*\.$''A good meal.' matches pattern '[A-Z][a-z]*( [a-z]+)*\.$'
```

The  first word starts with a capital, followed by zero or more lowercase  letters. Then, we enter a parenthetical that matches a single space  followed by a word of one or more lowercase letters. This entire  parenthetical is repeated zero or more times, and the pattern is  terminated with a period. There cannot be any other characters after the  period, as indicated by the `$` matching the end of string.

We've  seen many of the most basic patterns, but the regular expression  language supports many more. I spent my first few years using regular  expressions looking up the syntax every time I needed to do something.  It is worth bookmarking Python's documentation for the `re` module and reviewing it frequently. There are very few things that regular expressions cannot match, and they should be the first tool you reach for when parsing strings.

# Getting Information from Regular Expressions

Let's now focus on the Python side of things. The regular expression syntax is the furthest thing from object-oriented programming. However, Python's `re` module provides an object-oriented interface to enter the regular expression engine.

We've been checking whether the `re.match` function returns a valid object or not. If a pattern does not match, that function returns `None`. If it does match, however, it returns a useful object that we can introspect for information about the pattern.

So far, our regular expressions have answered questions such as, **does this string match this pattern?** Matching patterns is useful, but in many cases, a more interesting question is, **if this string matches this pattern, what is the value of a relevant substring?** If  you use groups to identify parts of the pattern that you want to  reference later, you can get them out of the match return value, as  illustrated in the next example:

```python
pattern = "^[a-zA-Z.]+@([a-z.]*\.[a-z]+)$" 
search_string = "some.user@example.com" 
match = re.match(pattern, search_string) 
 
if match: 
    domain = match.groups()[0] 
    print(domain) 
```

The specification describing  valid email addresses is extremely complicated, and the regular  expression that accurately matches all possibilities is obscenely long.  So, we cheated and made a simple regular expression that matches some  common email addresses; the point is that we want to access the domain  name (after the `@` sign) so we can connect to that address. This is done easily by wrapping that part of the pattern in parentheses and calling the `groups()` method on the object returned by `match`.

The `groups`  method returns a tuple of all the groups matched inside the pattern,  which you can index to access a specific value. The groups are ordered  from left to right. However, bear in mind that groups can be nested,  meaning you can have one or more groups inside another group. In this  case, the groups are returned in the order of their leftmost brackets,  so the outermost group will be returned before its inner matching  groups.

In addition to the `match` function, the `re` module provides a couple of other useful functions, `search` and `findall`. The `search`  function finds the first instance of a matching pattern, relaxing the  restriction that the pattern should start at the first letter of the  string. Note that you can get a similar effect by using `match` and putting a  `^.*`  character at the front of the pattern to match any characters between  the start of the string and the pattern you are looking for.

The `findall`  function behaves similarly to search, except that it finds all  non-overlapping instances of the matching pattern, not just the first  one. Basically, it finds the first match, then it resets the search to  the end of that matching string and finds the next one.

Instead of  returning a list of match objects, as you would expect, it returns a  list of matching strings, or tuples. Sometimes it's strings, sometimes  it's tuples. It's not a very good API at all! As with all bad APIs,  you'll have to memorize the differences and not rely on intuition. The  type of the return value depends on the number of bracketed groups  inside the regular expression:

- If there are no groups in the pattern, `re.findall` will return a list of strings, where each value is a complete substring from the source string that matches the pattern
- If there is exactly one group in the pattern, `re.findall` will return a list of strings where each value is the contents of that group
- If there are multiple groups in the pattern, `re.findall` will return a list of tuples where each tuple contains a value from a matching group, in order

info> When  you are designing function calls in your own Python libraries, try to  make the function always return a consistent data structure. It is often  good to design functions that can take arbitrary inputs and process  them, but the return value should not switch from a single value to a  list, or a list of values to a list of tuples depending on the input.  Let `re.findall` be a lesson!

The examples in the following interactive session will hopefully clarify the differences:

```python
>>> import re>>> re.findall('a.', 'abacadefagah')['ab', 'ac', 'ad', 'ag', 'ah']>>> re.findall('a(.)', 'abacadefagah')['b', 'c', 'd', 'g', 'h']>>> re.findall('(a)(.)', 'abacadefagah')[('a', 'b'), ('a', 'c'), ('a', 'd'), ('a', 'g'), ('a', 'h')]>>> re.findall('((a)(.))', 'abacadefagah')[('ab', 'a', 'b'), ('ac', 'a', 'c'), ('ad', 'a', 'd'), ('ag', 'a', 'g'), ('ah', 'a', 
'h')]
```

## Making Repeated Regular Expressions Efficient

Whenever you call one of the regular expression  methods, the engine has to convert the pattern string into an internal  structure that makes searching strings fast. This conversion takes a  non-trivial amount of time. If a regular expression pattern is going to  be reused multiple times (for example, inside a `for` or `while` loop), it would be better if this conversion step could be done only once.

This is possible with the `re.compile`  method. It returns an object-oriented version of the regular expression  that has been compiled down and has the methods we've explored (`match`, `search`, and `findall`) already, among others. We'll see examples of this in the case study.

This  has definitely been a condensed introduction to regular expressions. At  this point, we have a good feel for the basics and will recognize when  we need to do further research. If we have a string pattern-matching  problem, regular expressions will almost certainly be able to solve them  for us. However, we may need to look up new syntaxes in a more  comprehensive coverage of the topic. But now we know what to look for!  Let's move on to a completely different topic: filesystem paths.