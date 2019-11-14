The facade pattern is designed to provide a simple interface to a complex system of components. For complex tasks, we may need to interact with these objects directly, but there is often a **typical** usage for the system for which these complicated interactions aren't necessary. The facade pattern allows us to define a new object that encapsulates this typical usage of the system. Any time we want access to common functionality, we can use the single object's simplified interface. If another part of the project needs access to more complicated functionality, it is still able to interact with the system directly. The UML diagram for the facade pattern is really dependent on the subsystem, but in a cloudy way, it looks like this:

![img](https://static.packt-cdn.com/products/9781789615852/graphics/98d0a5f2-0392-4324-b0fa-0e71ed3628d6.png)

A facade is, in many ways, like an adapter. The primary difference is that a facade tries to abstract a simpler interface out of a complex one, while an adapter only tries to map one existing interface to another.

Let's write a simple facade for an email application. The low-level library for sending email in Python, as we saw in **Python Object-Oriented Shortcuts**, is quite complicated. The two libraries for receiving messages are even worse.

It  would be nice to have a simple class that allows us to send a single  email, and list the emails currently in the inbox on an IMAP or POP3  connection. To keep our example short, we'll stick with IMAP and SMTP:  two totally different subsystems that happen to deal with email. Our  facade performs only two tasks: sending an email to a specific address,  and checking the inbox on an IMAP connection. It makes some common  assumptions about the connection, such as that the host for both SMTP  and IMAP is at the same address, that the username and password for both  is the same, and that they use standard ports. This covers  the case for many email servers, but if a programmer needs more  flexibility, they can always bypass the facade and access the two  subsystems directly.

The class is initialized with the hostname of the email server, a username, and a password to log in:

```python
import smtplib 
import imaplib 
 
class EmailFacade: 
    def __init__(self, host, username, password): 
        self.host = host 
        self.username = username 
        self.password = password 
```

The `send_email` method formats the email address and message, and sends it using `smtplib`. This isn't a complicated task, but it requires quite a bit of fiddling to massage the **natural** input parameters that are passed into the facade to the correct format to enable `smtplib` to send the message, as follows:

```python
    def send_email(self, to_email, subject, message):
        if not "@" in self.username:
            from_email = "{0}@{1}".format(self.username, self.host)
        else:
            from_email = self.username
        message = (
            "From: {0}\r\n" "To: {1}\r\n" "Subject: {2}\r\n\r\n{3}"
        ).format(from_email, to_email, subject, message)

        smtp = smtplib.SMTP(self.host)
        smtp.login(self.username, self.password)
        smtp.sendmail(from_email, [to_email], message)
```

The `if` statement at the beginning of the method is catching whether or not the `username` is the entire **from** email address or just the part on the left-hand side of the `@` symbol; different hosts treat the login details differently.

Finally,  the code to get the messages currently in the inbox is a royal mess.  The IMAP protocol is painfully over-engineered, and the `imaplib` standard library is only a thin layer over the protocol. But we get to simplify it, as follows:

```python
    def get_inbox(self):
        mailbox = imaplib.IMAP4(self.host)
        mailbox.login(
            bytes(self.username, "utf8"), bytes(self.password, "utf8")
        )
        mailbox.select()
        x, data = mailbox.search(None, "ALL")
        messages = []
        for num in data[0].split():
            x, message = mailbox.fetch(num, "(RFC822)")
            messages.append(message[0][1])
        return messages
```

Now, if we add all this  together, we have a simple facade class that can send and receive  messages in a fairly straightforward manner; much simpler than if we had  to interact with these complex libraries directly.

Although it is rarely mentioned by name in the Python  community, the facade pattern is an integral part of the Python  ecosystem. Because Python emphasizes language readability, both the  language and its libraries tend to provide easy-to-comprehend interfaces  to complicated tasks. For example, `for` loops, `list` comprehensions, and generators are all facades into a more complicated iterator protocol. The `defaultdict` implementation is a facade that abstracts away annoying corner cases when a key doesn't exist in a dictionary. The third-party **requests**  library is a powerful facade over less readable libraries for HTTP  requests, which are themselves a facade over managing the text-based  HTTP protocol yourself.