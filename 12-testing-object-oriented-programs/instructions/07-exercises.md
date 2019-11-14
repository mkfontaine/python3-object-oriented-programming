Practice test-driven development. That is your first exercise. It's  easier to do this if you're starting a new project, but if you have  existing code you need to work on, you can start by writing tests for  each new feature you implement. This can become frustrating as you  become more enamored with automated tests. The old, untested code will  start to feel rigid and tightly coupled, and will become uncomfortable  to maintain; you'll start feeling like changes you make are breaking the  code and you have no way of knowing, for lack of tests. But if you  start small, adding tests to the code base improves it over time.

So, to get your feet wet with test-driven development, start a fresh project. Once you've started to appreciate the  benefits (you will) and realize that the time spent writing tests is  quickly regained in terms of more maintainable code, you'll want to  start writing tests for existing code. This is when you should start  doing it, not before. Writing tests for code that we **know** works  is boring. It is hard to get interested in the project until you  realize just how broken the code we thought was working really is.

Try writing the same set of tests using both the built-in `unittest` module and `pytest`. Which do you prefer? `unittest` is more similar to test frameworks in other languages, while `pytest` is arguably more Pythonic. Both allow us to write object-oriented tests and to test object-oriented programs with ease.

We used `pytest` in our case study, but we didn't touch on any features that wouldn't have been easily testable using `unittest`. Try adapting the tests to use test skipping or fixtures (an instance of `VignereCipher`  would be helpful). Try the various setup and teardown methods, and  compare their use to funcargs. Which feels more natural to you?

Try  running a coverage report on the tests you've written. Did you miss  testing any lines of code? Even if you have 100 percent coverage, have  you tested all the possible inputs? If you're doing test-driven  development, 100 percent coverage should follow quite naturally, as you  will write a test before the code that satisfies that test. However, if  writing tests for existing code, it is more likely that there will be  edge conditions that go untested.

Think carefully about the values that are somehow different, such as the following, for example:

- Empty lists when you expect full ones
- Negative numbers, zero, one, or infinity compared to positive integers
- Floats that don't round to an exact decimal place
- Strings when you expected numerals
- Unicode strings when you expected ASCII
- The ubiquitous `None` value when you expected something meaningful

If your tests cover such edge cases, your code will be in good shape.