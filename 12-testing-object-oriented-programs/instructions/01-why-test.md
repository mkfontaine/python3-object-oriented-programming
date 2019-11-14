Many programmers already know how important  it is to test their code. If you're among them, feel free to skim this  section. You'll find the next section–where we actually see how to  create tests in Python–much more scintillating. If you're not convinced  of the importance of testing, I promise that your code is broken, you  just don't know it. Read on!

Some people argue that testing is  more important in Python code because of its dynamic nature; compiled  languages such as Java and C++ are occasionally thought to be somehow **safer** because  they enforce type checking at compile time. However, Python tests  rarely check types. They check values. They make sure that the right  attributes have been set at the right time or that the sequence has the  right length, order, and values. These higher-level concepts  need to be tested in any language. The real reason Python programmers  test more than programmers of other languages is that it is so easy to  test in Python!

But why test? Do we really need to test? What if  we didn't test? To answer those questions, write a tic-tac-toe game from  scratch without any testing at all. Don't run it until it is completely  written, start to finish. Tic-tac-toe is fairly simple to implement if  you make both players human players (no artificial intelligence). You  don't even have to try to calculate who the winner is. Now run your  program. And fix all the errors. How many were there? I recorded eight in my tic-tac-toe implementation, and I'm not sure I caught them all. Did you?

We  need to test our code to make sure it works. Running the program, as we  just did, and fixing the errors is one crude form of testing. Python's  interactive interpreter and near-zero compile times makes it easy to  write a few lines of code and run the program to make sure those lines  are doing what is expected. But changing a few lines of code can affect  parts of the program that we haven't realized will be influenced by the  changes, and therefore neglect to test those parts. Furthermore, as a program grows, the number of paths that the interpreter can take through that code also grow, and it quickly becomes impossible to manually test all of them.

To  handle this, we write automated tests. These are programs that  automatically run certain inputs through other programs or parts of  programs. We can run these test programs in seconds and cover far more  potential input situations than one programmer would think to test every  time they change something.

There are four main reasons to write tests:

- To ensure that code is working the way the developer thinks it should
- To ensure that code continues working when we make changes
- To ensure that the developer understood the requirements
- To ensure that the code we are writing has a maintainable interface

The  first point really doesn't justify the time it takes to write a test;  we can test the code directly in the interactive interpreter in the same  time or less. But when we have to perform the same sequence of test  actions multiple times, it takes less time to automate those steps once  and then run them whenever necessary. It is a good idea to run tests  every time we change code, whether it is during initial development or  maintenance releases. When we have a comprehensive set of automated  tests, we can run them after code changes and know that we didn't  inadvertently break anything that was tested.

The last two of the preceding points are more  interesting. When we write tests for code, it helps us design the API,  interface, or pattern that code takes. Thus, if we misunderstood the  requirements, writing a test can help highlight that misunderstanding.  From the other side, if we're not certain how we want to design a class,  we can write a test that interacts with that class so we have an idea  of the most natural way to interface with it. In fact, it is often  beneficial to write the tests before we write the code we are testing.

# Test-Driven Development

**Write tests first** is the mantra of test-driven development. Test-driven development takes the **untested code is broken code** concept one step further and suggests  that only unwritten code should be untested. We don't write any code  until we have written the tests that will prove it works. The first time  we run a test it should fail, since the code hasn't been  written. Then, we write the code that ensures the test passes, then  write another test for the next segment of code.

Test-driven  development is fun; it allows us to build little puzzles to solve. Then,  we implement the code to solve those puzzles. Then, we make a more  complicated puzzle, and we write code that solves the new puzzle without  unsolving the previous one.

There are two goals to the  test-driven methodology. The first is to ensure that tests really get  written. It's so very easy, after we have written code, to say:

> **"Hmm, it seems to work. I don't have to write any tests for this. It was just a small change; nothing could have broken."**

If  the test is already written before we write the code, we will know  exactly when it works (because the test will pass), and we'll know in  the future if it is ever broken by a change we or someone else has made.

Secondly,  writing tests first forces us to consider exactly how the code will be  used. It tells us what methods objects need to have and how attributes  will be accessed. It helps us break up the initial problem into smaller,  testable problems, and then to recombine the tested solutions into larger, also tested, solutions. Writing tests can thus become a part of the design process. Often, when  we're writing a test for a new object, we discover anomalies in the  design that force us to consider new aspects of the software.

As a  concrete example, imagine writing code that uses an object-relational  mapper to store object properties in a database. It is common to use an  automatically assigned database ID in such objects. Our code might use  this ID for various purposes. If we are writing a test for such code,  before we write it, we may realize that our design is faulty because  objects do not have IDs assigned until they have been saved to the  database. If we want to manipulate an object without saving it in our  test, it will highlight this problem before we have written code based  on the faulty premise.

Testing makes software better. Writing tests before  we release the software makes it better before the end user sees or  purchases the buggy version (I have worked for companies that thrive on  the **users can test it** philosophy;  it's not a healthy business model). Writing tests before we write  software makes it better the first time it is written.