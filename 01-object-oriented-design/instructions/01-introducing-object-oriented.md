Everyone knows what an object is: a tangible  thing that we can sense, feel, and manipulate. The earliest objects we  interact with are typically baby toys. Wooden blocks, plastic shapes,  and over-sized puzzle pieces are common first objects. Babies learn  quickly that certain objects do certain things: bells ring, buttons are pressed, and levers are pulled.

The  definition of an object in software development is not terribly  different. Software objects may not be tangible things that you can pick  up, sense, or feel, but they are models of something that can do  certain things and have certain things done to them. Formally, an object  is a collection of **data** and associated **behaviors**.

So, knowing what an object is, what does it mean to be object-oriented? In the dictionary, **oriented** means **directed toward**.  So object-oriented means functionally directed toward modeling objects.  This is one of many techniques used for modeling complex systems. It is  defined by describing a collection of interacting objects via their  data and behavior.

If you've read any hype, you've probably come across the terms **object-oriented analysis**, **object-oriented design**, **object-oriented analysis and design**, and **object-oriented programming**. These are all highly related concepts under the general **object-oriented** umbrella.

In  fact, analysis, design, and programming are all stages of software  development. Calling them object-oriented simply specifies what level of  software development is being pursued.

**Object-oriented analysis** (**OOA**) is the process  of looking at a problem, system, or task (that somebody wants to turn  into an application) and identifying the objects and interactions  between those objects. The analysis stage is all about **what** needs to be done.

The  output of the analysis stage is a set of requirements. If we were to  complete the analysis stage in one step, we would have turned a task,  such as **I need a website**, into a set of requirements.  As an example, here or some requirements as to what a website visitor might need to do (**italic** represents actions, **bold** represents objects):

- **Review** our **history**
- **Apply** for **jobs**
- **Browse**, **compare**, and **order products**

In some ways, **analysis** is  a misnomer. The baby we discussed earlier doesn't analyze the blocks  and puzzle pieces. Instead, she explores her environment, manipulates  shapes, and sees where they might fit. A better turn of phrase might be **object-oriented exploration**.  In software development, the initial stages of analysis include  interviewing customers, studying their processes, and eliminating  possibilities.

**Object-oriented design** (**OOD**) is the process  of converting such requirements into an implementation specification.  The designer must name the objects, define the behaviors, and formally  specify which objects can activate specific behaviors on other objects.  The design stage is all about **how** things should be done.

The  output of the design stage is an implementation specification. If we  were to complete the design stage in a single step, we would have turned  the requirements defined during object-oriented analysis into a set of  classes and interfaces that could be implemented in (ideally) any  object-oriented programming language.

**Object-oriented programming** (**OOP**) is the process of converting this perfectly-defined design into a working program that does exactly what the CEO originally requested.

Yeah,  right! It would be lovely if the world met this ideal and we could  follow these stages one by one, in perfect order, like all the old  textbooks told us to. As usual, the real world is much murkier. No  matter how hard we try to separate these stages, we'll always find  things that need further analysis while we're designing. When we're  programming, we find features that need clarification in the design.

Most  twenty-first century development happens in an iterative development  model. In iterative development, a small part of the task is modeled,  designed, and programmed, and then the program is reviewed and expanded  to improve each feature and include new features in a series of short  development cycles.

The rest of this course is about object-oriented  programming, but in this chapter, we will cover the basic  object-oriented principles in the context of design. This allows us to  understand these (rather simple) concepts without having to argue with  software syntax or Python tracebacks.