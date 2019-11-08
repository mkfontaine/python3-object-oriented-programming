Let's try to tie everything we've learned together  with a larger example. We'll be developing an automated grading system  for programming assignments, similar to that employed at Dataquest or  Coursera. The system will need to provide a simple class-based interface  for course writers to create their assignments and should give a useful  error message if it does not fulfill that interface. The writers need  to be able to supply their lesson content and to write custom answer  checking code to make sure their students got the answer right. It will  also be nice for them to have access to the students' names to make the  content seem a little friendlier.

The grader itself will need to  keep track of which assignment the student is currently working on. A  student might make several attempts at an assignment before they get it  right. We want to keep track of the number of attempts so the course  authors can improve the content of the more difficult lessons.

Let's  start by defining the interface that the course authors will need to  use. Ideally, it will require the course authors to write a minimal  amount of extra code besides their lesson content and answer checking  code. Here is the simplest class I could come up with:

```python
class IntroToPython:
    def lesson(self):
        return f"""
            Hello {self.student}. define two variables,
            an integer named a with value 1
            and a string named b with value 'hello'

        """

    def check(self, code):
        return code == "a = 1\nb = 'hello'"
```

Admittedly, that particular course author may be a little naïve in how they do their answer checking. If you haven't seen the `f"""` syntax before, we'll cover it in detail in the **Strings and Serialization**.

We can start with an abstract base class that defines this interface, as follows:

```python
class Assignment(metaclass=abc.ABCMeta):
    @abc.abstractmethod
    def lesson(self, student):
        pass

    @abc.abstractmethod
    def check(self, code):
        pass

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Assignment:
            attrs = set(dir(C))
            if set(cls.__abstractmethods__) <= attrs:
                return True

        return NotImplemented
```

This ABC defines the two required abstract methods and provides the magic `__subclasshook__`  method to allow a class to be perceived as a subclass without having to  explicitly extend it (I usually just copy and paste this code. It isn't  worth memorizing.)

We can confirm that the `IntroToPython` class fulfills this interface using `issubclass(IntroToPython, Assignment)`, which should return `True`. Of course, we can explicitly extend the `Assignment` class if we prefer, as seen in this second assignment:

```python
class Statistics(Assignment):
    def lesson(self):
        return (
            "Good work so far, "
            + self.student
            + ". Now calculate the average of the numbers "
            + " 1, 5, 18, -3 and assign to a variable named 'avg'"
        )

    def check(self, code):
        import statistics

        code = "import statistics\n" + code

        local_vars = {}
        global_vars = {}
        exec(code, global_vars, local_vars)

        return local_vars.get("avg") == statistics.mean([1, 5, 18, -3])
```

This course author, unfortunately, is also rather naive. The `exec`  call will execute the student's code right inside the grading system,  giving them access to the entire system. Obviously, the first thing they  will do is hack the system to make their grades 100%. They probably  think that's easier than doing the assignments correctly!

Next, we'll create a class that manages how many attempts the student has made at a given assignment:

```python
class AssignmentGrader:
    def __init__(self, student, AssignmentClass):
        self.assignment = AssignmentClass()
        self.assignment.student = student
        self.attempts = 0
        self.correct_attempts = 0

    def check(self, code):
        self.attempts += 1
        result = self.assignment.check(code)
        if result:
            self.correct_attempts += 1

        return result

    def lesson(self):
        return self.assignment.lesson()
```

This class uses composition instead of inheritance. At first glance, it would make sense for these methods to exist on the `Assignment` superclass. That would eliminate the annoying `lesson`  method, which just proxies through to the same method on the assignment  object. It would certainly be possible to put all this logic directly  on the `Assignment` abstract base class, or even to have the ABC inherit from this `AssignmentGrader`  class. In fact, I would normally recommend that, but in this case, it  would force all course authors to explicitly extend the class, which  violates our request that content authoring be as simple as possible.

Finally, we can start to put together the `Grader` class, which is responsible for managing which assignments are available and which one each student is currently working on. The most interesting part is the register method:

```python
import uuid

class Grader:
    def __init__(self):
        self.student_graders = {}
        self.assignment_classes = {}

    def register(self, assignment_class):
        if not issubclass(assignment_class, Assignment):
            raise RuntimeError(
                "Your class does not have the right methods"
            )

        id = uuid.uuid4()
        self.assignment_classes[id] = assignment_class
        return id
```

This code block includes the initializer, which includes two dictionaries we'll discuss in a minute. The `register` method is a bit complex, so we'll dissect it thoroughly.

The first odd thing is the parameter this method accepts: `assignment_class`.  This parameter is intended to be an actual class, not an instance of  the class. Remember, classes are objects, too, and can be passed around  like other classes. Given the `IntroToPython` class we defined earlier, we might register it without instantiating it, as follows:

```python
from grader import Grader
from lessons import IntroToPython, Statistics

grader = Grader()
itp_id = grader.register(IntroToPython)
```

The method first checks whether that class is a subclass of the `Assignment` class. Of course, we implemented a custom `__subclasshook__` method, so this includes classes that do not explicitly subclass `Assignment`.  The naming is, perhaps, a bit deceitful! If it doesn't have the two  required methods, it raises an exception. Exceptions are a topic we'll  cover in detail in the next lesson; for now, just assume that it makes  the program get angry and quit.

Then, we generate a random identifier to represent that specific assignment. We store the `assignment_class`  in a dictionary indexed by that ID, and return the ID so that the  calling code can look that assignment up in the future. Presumably,  another object would then place that ID in a course syllabus of some  sort so students do the assignments in order, but we won't be doing that  for this part of the project.

info> The `uuid`  function returns a specially formatted string called a universally  unique identifier, also known as a globally unique identifier. It  essentially represents an extremely large random number that is almost,  but not quite, impossible to conflict with another similarly generated  identifier. It is a great, quick, and clean way to create an arbitrary  ID to keep track of items.

Next up, we have the `start_assignment`  function, which allows a student to start working on an assignment  given the ID of that assignment. All it does is construct an instance of  the `AssignmentGrader` class we defined earlier and plop it in a dictionary stored on the `Grader` class, as follows:

```python
    def start_assignment(self, student, id):
        self.student_graders[student] = AssignmentGrader(
            student, self.assignment_classes[id]
        )
```

After that, we write a couple of proxy  methods that get the lesson or check the code for whatever assignment  the student is currently working on:

```python
    def get_lesson(self, student):
        assignment = self.student_graders[student]
        return assignment.lesson()

    def check_assignment(self, student, code):
        assignment = self.student_graders[student]
        return assignment.check(code)
```

Finally, we  create a method that gives a summary of a student's current assignment  progress. It looks up the assignment object and creates a formatted  string with all the information we have about that student:

```python
    def assignment_summary(self, student):
        grader = self.student_graders[student]
        return f"""
        {student}'s attempts at {grader.assignment.__class__.__name__}:

        attempts: {grader.attempts}
        correct: {grader.correct_attempts}

        passed: {grader.correct_attempts > 0}
        """
```

And that's it. You'll notice that this  case study does not use a ton of inheritance, which may seem a bit odd  given the topic of the chapter, but duck typing is very prevalent. It is  quite common for Python programs to be designed with inheritance that  gets simplified into more versatile constructs as it is iterated on. As another example, I originally defined the `AssignmentGrader`  as an inheritance relationship, but realized halfway through that it  would be better to use composition, for the reasons outlined previously.

Here's a bit of test code that shows all these objects connected together:

```python
grader = Grader()
itp_id = grader.register(IntroToPython)
stat_id = grader.register(Statistics)

grader.start_assignment("Tammy", itp_id)
print("Tammy's Lesson:", grader.get_lesson("Tammy"))
print(
    "Tammy's check:",
    grader.check_assignment("Tammy", "a = 1 ; b = 'hello'"),
)
print(
    "Tammy's other check:",
    grader.check_assignment("Tammy", "a = 1\nb = 'hello'"),
)

print(grader.assignment_summary("Tammy"))

grader.start_assignment("Tammy", stat_id)
print("Tammy's Lesson:", grader.get_lesson("Tammy"))
print("Tammy's check:", grader.check_assignment("Tammy", "avg=5.25"))
print(
    "Tammy's other check:",
    grader.check_assignment(
        "Tammy", "avg = statistics.mean([1, 5, 18, -3])"
    ),
)

print(grader.assignment_summary("Tammy"))
```