To tie together some of the principles presented in this lesson,  let's build a mailing list manager. The manager will keep track of email  addresses categorized  into named groups. When it's time to send a message, we can pick a  group and send the message to all email addresses assigned to that  group.

Now, before we start working on this project, we ought to  have a safe way to test it, without sending emails to a bunch of real  people. Luckily, Python has our back here; like the test HTTP server, it  has a built-in **Simple Mail Transfer Protocol** (**SMTP**) server that  we can instruct to capture any messages we send without actually  sending them. We can run the server with the following command:

```bash
$python -m smtpd -n -c DebuggingServer localhost:1025
```

Running  this command at command prompt will start an SMTP server running on  port 1025 on the local machine. But we've instructed it to use the `DebuggingServer`  class (this class comes with the built-in SMTP module), which, instead  of sending mails to the intended recipients, simply prints them on the  terminal screen as it receives them.

Now, before writing our  mailing list, let's write some code that actually sends mail. Of course,  Python supports this in the standard library, too, but it's a bit of an  odd interface, so we'll write a new function to wrap it all cleanly, as  can be seen in the following code snippet:

```python
import smtplib
from email.mime.text import MIMEText


def send_email(
    subject,
    message,
    from_addr,
    *to_addrs,
    host="localhost",
    port=1025,
    **headers
):

    email = MIMEText(message)
    email["Subject"] = subject
    email["From"] = from_addr
    for header, value in headers.items():
        email[header] = value

    sender = smtplib.SMTP(host, port)
    for addr in to_addrs:
        del email["To"]
        email["To"] = addr
        sender.sendmail(from_addr, addr, email.as_string())
    sender.quit()
```

We won't cover the code inside this method too thoroughly; the documentation in the standard library can give you all the information you need to use the `smtplib` and `email` modules effectively.

We've  used both variable argument and keyword argument syntax in the function  call. The variable argument list allows us to supply a single string in  the default case of having a single `to`  address, as well as permitting multiple addresses to be supplied if  required. Any extra keyword arguments are mapped to email headers. This  is an exciting use of variable arguments and keyword arguments, but it's  not really a great interface for the person calling the function. In  fact, it makes many things the programmer will want to do impossible.

The headers passed into the function represent auxiliary headers that can be attached to a method. Such headers might include `Reply-To`, `Return-Path`, or **X-pretty-much-anything**. But in order to be a valid identifier in Python, a name cannot include the `-` character. In general, that character represents subtraction. So, it's not possible to call a function with `Reply-To``=``my@email.com`. As often happens, it appears we were too eager to use keyword arguments because they are a shiny new tool we just learned.

We'll  have to change the argument to a normal dictionary; this will work  because any string can be used as a key in a dictionary. By default,  we'd want this dictionary to be empty, but we can't make the default  parameter an empty dictionary. So, we'll have to make the default  argument `None`, and then set up the dictionary at the beginning of the method, as follows:

```python
def send_email(subject, message, from_addr, *to_addrs, 
        host="localhost", port=1025, headers=None): 
 
    headers = headers if headers else {}
```

If we have our debugging SMTP server running in one terminal, we can test this code in a Python interpreter:

```python
>>> send_email("A model subject", "The message contents", "from@example.com", "to1@example.com", "to2@example.com")
```

Then, if we check the output from the debugging SMTP server, we get the following:

```python
---------- MESSAGE FOLLOWS ----------Content-Type: text/plain; charset="us-ascii"MIME-Version: 1.0Content-Transfer-Encoding: 7bitSubject: A model subjectFrom: from@example.comTo: to1@example.comX-Peer: 127.0.0.1The message contents------------ END MESSAGE ---------------------- MESSAGE FOLLOWS ----------Content-Type: text/plain; charset="us-ascii"MIME-Version: 1.0Content-Transfer-Encoding: 7bitSubject: A model subjectFrom: from@example.comTo: to2@example.comX-Peer: 127.0.0.1The message contents------------ END MESSAGE ------------
```

Excellent, it has **sent** our  email to the two expected addresses with subject and message contents  included. Now that we can send messages, let's work on the email group  management system. We'll need an object that somehow matches email  addresses with the groups they are in. Since this is a many-to-many  relationship (any one email address can be in multiple groups; any one group  can be associated with multiple email addresses), none of the data  structures we've studied seem ideal. We could try a dictionary of group  names matched to a list of associated email addresses, but that would  duplicate email addresses. We could also try a dictionary of email  addresses matched to groups, resulting in a duplication of groups.  Neither seems optimal. For fun, let's try this latter version, even  though intuition tells me the groups to email address solution would be  more straightforward.

Since the values in our dictionary will always be collections of unique email addresses, we can store them in a `set` container. We can use `defaultdict` to ensure that there is always a `set` container available for each key, demonstrated as follows:

```python
from collections import defaultdict


class MailingList:
    """Manage groups of e-mail addresses for sending e-mails."""

    def __init__(self):
        self.email_map = defaultdict(set)

    def add_to_group(self, email, group):
        self.email_map[email].add(group)
```

Now, let's  add a method that allows us to collect all the email addresses in one  or more groups. This can be done by converting the list of groups to a  set:

```python
def emails_in_groups(self, *groups): groups = set(groups) emails = set() for e, g in self.email_map.items(): if g & groups: emails.add(e) return emails 
```

First, look at what we're iterating over: `self.email_map.items()`.  This method, of course, returns a tuple of key-value pairs for each  item in the dictionary. The values are sets of strings representing the  groups. We split these into two variables named `e` and `g`, short for email  and groups. We add the email address to the set of return values only  if the passed-in groups intersect with the email address groups. The `g``&``groups` syntax is a shortcut for `g.intersection(groups)`; the `set` class does this by implementing the special `__and__` method to call `intersection`.

info> This code could be made a wee bit more concise using a set comprehension, which we'll discuss in **The Iterator Pattern**.

Now, with these building blocks, we can trivially add a method to our `MailingList` class that sends messages to specific groups:

```python
    def send_mailing(
        self, subject, message, from_addr, *groups, headers=None
    ):
        emails = self.emails_in_groups(*groups)
        send_email(
            subject, message, from_addr, *emails, headers=headers
        )
```

This function relies on variable argument  lists. As input, it takes a list of groups as variable arguments. It  gets the list of emails for the specified groups and passes those as  variable arguments into `send_email`, along with other arguments that were passed into this method.

The program can be tested by ensuring that the SMTP debugging server is running in one command prompt, and, in a second prompt, loading the code using the following:

```bash
$python -i mailing_list.py
```

Create a `MailingList` object with the help of the following command:

```bash
>>> m = MailingList()
```

Then, create a few fake email addresses and groups, along the lines of:

```python
>>> m.add_to_group("friend1@example.com", "friends")>>> m.add_to_group("friend2@example.com", "friends")>>> m.add_to_group("family1@example.com", "family")>>> m.add_to_group("pro1@example.com", "professional")
```

Finally, use a command like this to send emails to specific groups:

```python
>>> m.send_mailing("A Party","Friends and family only: a party", "me@example.com", "friends","family", headers={"Reply-To": "me2@example.com"})
```

Emails to each of the addresses in the specified groups should show up in the console on the SMTP server.

The  mailing list works fine as it is, but it's kind of useless; as soon as  we exit the program, our database of information is lost. Let's modify  it to add a couple of methods to load and save the list of email groups  from and to a file.

In general, when storing structured data on  disk, it is a good idea to put a lot of thought into how it is stored.  One of the reasons myriad database systems exist is that if someone else  has put this thought into how data is stored, you don't have to. We'll  be looking at some data serialization mechanisms in the next chapter,  but for this example, let's keep it simple and go with the first  solution that could possibly work.

The data format I have in mind is to store  each email address followed by a space, followed by a comma-separated  list of groups. This format seems reasonable, and we're going to go with  it because data formatting isn't the topic of this chapter. However, to  illustrate just why you need to think hard about how you format data on  disk, let's highlight a few problems with the format.

First, the  space character is technically legal in email addresses. Most email  providers prohibit it (with good reason), but the specification defining  email addresses says an email can contain a space if it is in quotation  marks. If we are to use a space as a sentinel in our data format, we  should technically be able to differentiate between that space and a  space that is part of an email. We're going to pretend this isn't true,  for simplicity's sake, but real-life data encoding is full of stupid  issues like this.

Second, consider the comma-separated list of  groups. What happens if someone decides to put a comma in a group name?  If we decide to make commas illegal in group names, we should add  validation to enforce such naming in our`add_to_group`  method. For pedagogical clarity, we'll ignore this problem too.  Finally, there are many security implications we need to consider: can  someone get themselves into the wrong group by putting a fake comma in  their email address? What does the parser do if it encounters an invalid  file?

The takeaway from this discussion is to try to use a data  storage method that has been field tested, rather than designing our own  data serialization protocols. There are a ton of bizarre edge cases you  might overlook, and it's better to use code that has already  encountered and fixed those edge cases.

But forget that. Let's just write some basic code that uses an unhealthy dose of wishful thinking to pretend this simple data format is safe, demonstrated as follows:

```python
email1@mydomain.com group1,group2email2@mydomain.com group2,group3
```

The code to do this is as follows:

```python
    def save(self):
        with open(self.data_file, "w") as file:
            for email, groups in self.email_map.items():
                file.write("{} {}\n".format(email, ",".join(groups)))

    def load(self):
        self.email_map = defaultdict(set)
        with suppress(IOError):
            with open(self.data_file) as file:
                for line in file:
                    email, groups = line.strip().split(" ")
                    groups = set(groups.split(","))
                    self.email_map[email] = groups
```

In the `save`  method, we open the file in a context manager and write the file as a  formatted string. Remember the newline character; Python doesn't add  that for us. The `load` method first resets the dictionary (in case it contains data from a previous call to `load`). It adds a call to the standard library `suppress` context manager, available as `from contextlib import suppress`.  This context manager catches any I/O Errors and ignores them. Not the  best error handling, but it's prettier than try...finally...pass.

Then, the load method uses the `for`...`in` syntax, which loops over each line in the file. Again, the newline character is included in the line variable, so we have to call `.strip()` to take it off. We'll learn more about such string manipulation in the next chapter.

Before using these methods, we need to make sure the object has a `self.data_file` attribute, which can be done by modifying `__init__` as follows:

```python
    def __init__(self, data_file): 
        self.data_file = data_file 
        self.email_map = defaultdict(set) 
```

We can test these two methods in the interpreter as follows:

```python
>>> m = MailingList('addresses.db')>>> m.add_to_group('friend1@example.com', 'friends')>>> m.add_to_group('family1@example.com', 'friends')>>> m.add_to_group('family1@example.com', 'family')>>> m.save()
```

The resulting `addresses.db` file contains the following lines, as expected:

```python
friend1@example.com friendsfamily1@example.com friends,family
```

We can also load this data back into a `MailingList` object successfully:

```python
>>> m = MailingList('addresses.db')>>> m.email_mapdefaultdict(<class 'set'>, {})>>> m.load()>>> m.email_mapdefaultdict(<class 'set'>, {'friend2@example.com': {'friends\n'}, 
'family1@example.com': {'family\n'}, 'friend1@example.com': {'friends\n'}})
```

As you can see, I forgot to add the `load` command, and it might be easy to forget the `save` command as well. To make this a little easier for anyone who wants to use our `MailingList` API in their own code, let's provide the methods to support a context manager:

```python
    def __enter__(self): 
        self.load() 
        return self 
 
    def __exit__(self, type, value, tb): 
        self.save() 
```

These simple methods just  delegate their work to load and save, but we can now write code like  this in the interactive interpreter and know that all the previously  stored addresses were loaded on our behalf, and that the whole list will  be saved to the file when we are done:

```python
>>> with MailingList('addresses.db') as ml:...    ml.add_to_group('friend2@example.com', 'friends')...    ml.send_mailing("What's up", "hey friends, how's it going", 'me@example.com', 
       'friends')
```