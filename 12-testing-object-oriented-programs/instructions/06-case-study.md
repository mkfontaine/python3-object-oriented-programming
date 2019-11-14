Let's walk through test-driven development by writing  a small, tested, cryptography application. Don't worry–you won't need  to understand the mathematics behind complicated modern encryption  algorithms such as AES or RSA. Instead, we'll be implementing a  sixteenth-century algorithm known as the Vigenère cipher. The  application simply needs to be able to encode and decode a message,  given an encoding keyword, using this cipher.

info> If you want a deep dive into how the RSA algorithm works, I wrote one on my blog [here](<https://dusty.phillips.codes/>).

First,  we need to understand how the cipher works if we apply it manually  (without a computer). We start with a table like the following one:

```markdown
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z 
B C D E F G H I J K L M N O P Q R S T U V W X Y Z A 
C D E F G H I J K L M N O P Q R S T U V W X Y Z A B 
D E F G H I J K L M N O P Q R S T U V W X Y Z A B C 
E F G H I J K L M N O P Q R S T U V W X Y Z A B C D 
F G H I J K L M N O P Q R S T U V W X Y Z A B C D E 
G H I J K L M N O P Q R S T U V W X Y Z A B C D E F 
H I J K L M N O P Q R S T U V W X Y Z A B C D E F G 
I J K L M N O P Q R S T U V W X Y Z A B C D E F G H 
J K L M N O P Q R S T U V W X Y Z A B C D E F G H I 
K L M N O P Q R S T U V W X Y Z A B C D E F G H I J 
L M N O P Q R S T U V W X Y Z A B C D E F G H I J K 
M N O P Q R S T U V W X Y Z A B C D E F G H I J K L 
N O P Q R S T U V W X Y Z A B C D E F G H I J K L M 
O P Q R S T U V W X Y Z A B C D E F G H I J K L M N 
P Q R S T U V W X Y Z A B C D E F G H I J K L M N O 
Q R S T U V W X Y Z A B C D E F G H I J K L M N O P 
R S T U V W X Y Z A B C D E F G H I J K L M N O P Q 
S T U V W X Y Z A B C D E F G H I J K L M N O P Q R 
T U V W X Y Z A B C D E F G H I J K L M N O P Q R S 
U V W X Y Z A B C D E F G H I J K L M N O P Q R S T 
V W X Y Z A B C D E F G H I J K L M N O P Q R S T U 
W X Y Z A B C D E F G H I J K L M N O P Q R S T U V 
X Y Z A B C D E F G H I J K L M N O P Q R S T U V W 
Y Z A B C D E F G H I J K L M N O P Q R S T U V W X 
Z A B C D E F G H I J K L M N O P Q R S T U V W X Y
```

Given a keyword, TRAIN, we can encode the message ENCODED IN PYTHON as follows:

1. Repeat the keyword and message together, such that it is easy to map letters from one to the other:

```markdown
E N C O D E D I N P Y T H O N
T R A I N T R A I N T R A I N
```

2. For each letter in the plaintext, find the row that begins with that letter in the table.
3. Find the column with the letter associated with the keyword letter for the chosen plaintext letter.
4. The encoded character is at the intersection of this row and column.

For  example, the row starting with E intersects the column starting with T  at character X. So, the first letter in the ciphertext is X. The row  starting with N intersects the column starting with R at character E,  leading to the ciphertext XE. C intersects A at C, and O intersects I at  W. D and N map to Q, while E and T map to X. The full encoded message  is XECWQXUIVCRKHWA.

Decoding follows the opposite procedure.  First, find the row with the character for the shared keyword (the T  row), then find the location in that row where the encoded character  (the X) is located. The plaintext character is at the top of the column for that row (the E).

# Implementing It

Our program will need an `encode` method that takes a keyword and plaintext and returns the ciphertext, and a `decode` method that accepts a keyword and ciphertext and returns the original message.

But rather than just writing those methods, let's follow a test-driven development strategy. We'll be using `pytest` for our unit testing. We need an `encode` method, and we know what it has to do; let's write a test for that method first, as follows:

```python
def test_encode():
    cipher = VigenereCipher("TRAIN")
    encoded = cipher.encode("ENCODEDINPYTHON")
    assert encoded == "XECWQXUIVCRKHWA"
```

This test fails, naturally, because we aren't importing a `VigenereCipher` class anywhere. Let's create a new module to hold that class.

Let's start with the following `VigenereCipher` class:

```python
class VigenereCipher:
    def __init__(self, keyword):
        self.keyword = keyword

    def encode(self, plaintext):
        return "XECWQXUIVCRKHWA"
```

If we add a `from` `vigenere_cipher` `import` `VigenereCipher` line to the top of our test class and run `pytest`, the preceding test will pass! We've finished our first test-driven development cycle.

This  may seem like a ridiculously silly thing to test, but it's actually  verifying a lot. The first time I implemented it, I misspelled cipher as **cypher** in  the class name. Even my basic unit test helped catch a bug. Even so,  returning a hardcoded string is obviously not the most sensible  implementation of a cipher class, so let's add a second test, as  follows:

```python
def test_encode_character(): 
    cipher = VigenereCipher("TRAIN") 
    encoded = cipher.encode("E") 
    assert encoded == "X" 
```

Ah, now that test will  fail. It looks like we're going to have to work harder. But I just  thought of something: what if someone tries to encode a string with  spaces or lowercase characters? Before we start implementing the  encoding, let's add some tests for these cases, so we don't forget them.  The expected behavior will be to remove spaces, and to convert  lowercase letters to capitals, as follows:

```python
def test_encode_spaces(): 
    cipher = VigenereCipher("TRAIN") 
    encoded = cipher.encode("ENCODED IN PYTHON") 
    assert encoded == "XECWQXUIVCRKHWA" 
 
def test_encode_lowercase(): 
    cipher = VigenereCipher("TRain") 
    encoded = cipher.encode("encoded in Python") 
    assert encoded == "XECWQXUIVCRKHWA" 
```

If we run  the new test suite, we find that the new tests pass (they expect the  same hardcoded string). But they ought to fail later if we forget to  account for these cases.

Now that we have some test cases, let's  think about how to implement our encoding algorithm. Writing code to use  a table like we used in the earlier manual algorithm is possible, but  seems complicated, considering that each row is just an alphabet rotated  by an offset number of characters. It turns out (I asked Wikipedia)  that we can use modular arithmetic to combine the characters instead of doing a table lookup.

Given  plaintext and keyword characters, if we convert the two letters to  their numerical values (according to their position in the alphabet,  with A being 0 and Z being 25), add them together, and take the  remainder mod 26, we get the ciphertext character! This is a  straightforward calculation, but since it happens on a  character-by-character basis, we should probably put it in its own  function. Before we do that, then, we should write a test for the new  function, as follows:

```python
from vigenere_cipher import combine_character 
def test_combine_character(): 
    assert combine_character("E", "T") == "X" 
    assert combine_character("N", "R") == "E" 
```

Now  we can write the code to make this function work. In all honesty, I had  to run the test several times before I got this function completely  correct. First, I accidentally returned an integer, and then I forgot to  shift the character back up to the normal ASCII scale from the  zero-based scale. Having the test available made it easy to test and  debug these errors. This is another bonus of test-driven development. The final, working version of the code looks like the following:

```python
def combine_character(plain, keyword): 
    plain = plain.upper() 
    keyword = keyword.upper() 
    plain_num = ord(plain) - ord('A') 
    keyword_num = ord(keyword) - ord('A') 
    return chr(ord('A') + (plain_num + keyword_num) % 26) 
```

Now that `combine_characters` is tested, I thought we'd be ready to implement our `encode`  function. However, the first thing we want inside that function is a  repeating version of the keyword string that is as long as the  plaintext. Let's implement a function for that first. Oops, I mean let's  implement the test first, as follows:

```python
def test_extend_keyword(): cipher = VigenereCipher("TRAIN") extended = cipher.extend_keyword(16) assert extended == "TRAINTRAINTRAINT" 
```

Before writing this test, I expected to write `extend_keyword`  as a standalone function that accepted a keyword and an integer. But as  I started drafting the test, I realized it made more sense to use it as  a helper method on the `VigenereCipher` class so it could access the `self.keyword`  attribute. This shows how test-driven development can help design more  sensible APIs. The following is the method implementation:

```python
    def extend_keyword(self, number):
        repeats = number // len(self.keyword) + 1
        return (self.keyword * repeats)[:number]
```

Once again, this took a few runs of the test to get right. I ended up adding an amended copy of the test, one with fifteen and one with sixteen letters, to make sure it works if the integer division has an even number.

Now we're finally ready to write our `encode` method, as follows:

```python
    def encode(self, plaintext): 
        cipher = [] 
        keyword = self.extend_keyword(len(plaintext)) 
        for p,k in zip(plaintext, keyword): 
            cipher.append(combine_character(p,k)) 
        return "".join(cipher) 
```

That looks correct. Our test suite should pass now, right?

Actually,  if we run it, we'll find that two tests are still failing. The  previously failing encode test is actually passing, but we totally  forgot about the spaces and lowercase characters! It is a good thing we  wrote those tests to remind us. We'll have to add the following line at  the beginning of the method:

```python
        plaintext = plaintext.replace(" ", "").upper() 
```



info> If  we have an idea about a corner case in the middle of implementing  something, we can create a test describing that idea. We don't even have  to implement the test; we can just run `assert False`  to remind us to implement it later. The failing test will never let us  forget the corner case and it can't be ignored as easily as a ticket in  an issue tracker. If it takes a while to get around to fixing the  implementation, we can mark the test as an expected failure.

Now all the tests pass successfully. This chapter is pretty long, so we'll condense the examples for decoding. The following are a couple of tests:

```python
def test_separate_character(): 
    assert separate_character("X", "T") == "E" 
    assert separate_character("E", "R") == "N" 
 
def test_decode(): 
    cipher = VigenereCipher("TRAIN") 
    decoded = cipher.decode("XECWQXUIVCRKHWA") 
    assert decoded == "ENCODEDINPYTHON" 
```

And the following is the `separate_character` function:

```python
def separate_character(cypher, keyword): 
    cypher = cypher.upper() 
    keyword = keyword.upper() 
    cypher_num = ord(cypher) - ord('A') 
    keyword_num = ord(keyword) - ord('A') 
    return chr(ord('A') + (cypher_num - keyword_num) % 26) 
```

Now we can add the `decode`method:

```python
    def decode(self, ciphertext): 
        plain = [] 
        keyword = self.extend_keyword(len(ciphertext)) 
        for p,k in zip(ciphertext, keyword): 
            plain.append(separate_character(p,k)) 
        return "".join(plain) 
```

These methods have a  lot of similarity to those used for encoding. The great thing about  having all these tests written and passing is that we can now go back  and modify our code, knowing it is still safely passing the tests. For  example, if we replace our existing `encode` and `decode` methods with the following refactored methods, our tests still pass:

```python
    def _code(self, text, combine_func): 
        text = text.replace(" ", "").upper() 
        combined = [] 
        keyword = self.extend_keyword(len(text)) 
        for p,k in zip(text, keyword): 
            combined.append(combine_func(p,k)) 
        return "".join(combined) 
 
    def encode(self, plaintext): 
        return self._code(plaintext, combine_character) 
 
    def decode(self, ciphertext): 
        return self._code(ciphertext, separate_character) 
```

This  is the final benefit of test-driven development, and the most  important. Once the tests are written, we can improve our code as much  as we like and be confident that our changes didn't break anything we  have been testing for. Furthermore, we know exactly when our refactor is  finished: when the tests all pass.

Of course, our tests may not  comprehensively test everything we need them to; maintenance or code  refactoring can still cause undiagnosed bugs that don't show up in  testing. Automated tests are not foolproof. If bugs do occur, however,  it is still possible to follow a test-driven plan, as follows:

1. Write a test (or multiple tests) that duplicates or **proves** that the bug in question is occurring. This will, of course, fail.
2. Then  write the code to make the tests stop failing. If the tests were  comprehensive, the bug will be fixed, and we will know if it ever  happens again, as soon as we run the test suite.

Finally, we can try to determine how well our tests operate on this code. With the `pytest` coverage plugin installed, `pytest -coverage-report=report`  tells us that our test suite has 100 percent code coverage. This is a  great statistic, but we shouldn't get too cocky about it. Our code  hasn't been tested when encoding messages that have numbers, and its  behavior with such inputs is thus undefined.