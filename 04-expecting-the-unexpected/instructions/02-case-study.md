We've been looking at the use and handling of exceptions  at a fairly low level of detail—syntax and definitions. This case study  will help tie it all in with our previous chapters so we can see how  exceptions are used in the larger context of objects, inheritance, and  modules.

Today, we'll be designing a simple central authentication  and authorization system. The entire system will be placed in one  module, and other code will be able to query that module object for  authentication and authorization purposes. We should admit, from the  start, that we aren't security experts, and that the system we are  designing may be full of security holes. Our purpose is to study  exceptions, not to secure a system. It will be sufficient, however, for a  basic login and permission system that other code can interact with.  Later, if that other code needs to be made more secure, we can have a  security or cryptography expert review or rewrite our module, preferably  without changing the API.

Authentication is the process of  ensuring a user is really the person they say they are. We'll follow the  lead of common web systems today, which use a username and private  password combination. Other methods of authentication include voice  recognition, fingerprint or retinal scanners, and identification cards.

Authorization,  on the other hand, is all about determining whether a given  (authenticated) user is permitted to perform a specific action. We'll  create a basic permission list system that stores a list of the specific  people allowed to perform each action.

In addition, we'll add  some administrative features to allow new users to be added to the  system. For brevity, we'll leave out editing of passwords or changing of  permissions once they've been added, but these (highly necessary) features can certainly be added in the future.

There's a simple analysis; now let's proceed with design. We're obviously going to need a `User`  class that stores the username and an encrypted password. This class  will also allow a user to log in by checking whether a supplied password  is valid. We probably won't need a `Permission` class, as those can just be strings mapped to a list of users using a dictionary. We should have a central `Authenticator` class that handles user management and logging in or out. The last piece of the puzzle is an `Authorizor`  class that deals with permissions and checking whether a user can  perform an activity. We'll provide a single instance of each of these  classes in the `auth` module so that other  modules can use this central mechanism for all their authentication and  authorization needs. Of course, if they want to instantiate private  instances of these classes, for non-central authorization activities,  they are free to do so.

We'll also be defining several exceptions as we go along. We'll start with a special `AuthException` base class that accepts a `username` and optional `user` object as parameters; most of our self-defined exceptions will inherit from this one.

Let's build the `User`  class first; it seems simple enough. A new user can be initialized with  a username and password. The password will be stored encrypted to  reduce the chances of its being stolen. We'll also need a `check_password` method to test whether a supplied password is the correct one. Here is the class in full:

```python
import hashlib


class User:
    def __init__(self, username, password):
        """Create a new user object. The password
        will be encrypted before storing."""
        self.username = username
        self.password = self._encrypt_pw(password)
        self.is_logged_in = False

    def _encrypt_pw(self, password):
        """Encrypt the password with the username and return
        the sha digest."""
        hash_string = self.username + password
        hash_string = hash_string.encode("utf8")
        return hashlib.sha256(hash_string).hexdigest()

    def check_password(self, password):
        """Return True if the password is valid for this
        user, false otherwise."""
        encrypted = self._encrypt_pw(password)
        return encrypted == self.password
```

Since the code for encrypting a password is required in both `__init__` and `check_password`,  we pull it out to its own method. This way, it only needs to be changed  in one place if someone realizes it is insecure and needs improvement.  This class could easily be extended to include mandatory or optional  personal details, such as names, contact information, and birth dates.

Before we write code to add users (which will happen in the as-yet undefined `Authenticator` class), we should examine some use cases. If all goes well, we can add a user with a username and password; the `User`  object is created and inserted into a dictionary. But in what ways can  all not go well? Well, clearly we don't want to add a user with a  username that already exists in the dictionary. If we did so, we'd  overwrite an existing user's data and the new user might have access to  that user's privileges. So, we'll need a `UsernameAlreadyExists` exception. Also, for security's sake, we should probably raise an exception if the password is too short. Both of these exceptions will extend `AuthException`, which we mentioned earlier. So, before writing the `Authenticator` class, let's define these three exception classes:

```python
class AuthException(Exception): 
    def __init__(self, username, user=None): 
        super().__init__(username, user) 
        self.username = username 
        self.user = user 
 
class UsernameAlreadyExists(AuthException): 
    pass 
 
class PasswordTooShort(AuthException): 
    pass 
```

The `AuthException` requires a username and has an optional user parameter. This second parameter should be an instance of the `User`  class associated with that username. The two specific exceptions we're  defining simply need to inform the calling class of an exceptional  circumstance, so we don't need to add any extra methods to them.

Now let's start on the `Authenticator`  class. It can simply be a mapping of usernames to user objects, so  we'll start with a dictionary in the initialization function. The method  for adding a user needs to check the two conditions (password length  and previously existing users) before creating a new `User` instance and adding it to the dictionary:

```python
class Authenticator:
    def __init__(self):
        """Construct an authenticator to manage
        users logging in and out."""
        self.users = {}

    def add_user(self, username, password):
        if username in self.users:
            raise UsernameAlreadyExists(username)
        if len(password) < 6:
            raise PasswordTooShort(username)
        self.users[username] = User(username, password)
```

We could, of course, extend the password validation to raise exceptions for passwords that are too easy to crack in other ways, if we desired. Now let's prepare the `login` method. If we weren't thinking about exceptions just now, we might just want the method to return `True` or `False`,  depending on whether the login was successful or not. But we are  thinking about exceptions, and this could be a good place to use them  for a not-so-exceptional circumstance. We could raise different  exceptions, for example, if the username does not exist or the password  does not match. This will allow anyone trying to log a user in to  elegantly handle the situation using a `try`/`except`/`else` clause. So, first we add these new exceptions:

```python
class InvalidUsername(AuthException): 
    pass 
 
class InvalidPassword(AuthException): 
    pass 
```

Then we can define a simple `login` method to our `Authenticator` class that raises these exceptions if necessary. If not, it flags the `user` as logged in and returns the following:

```python
    def login(self, username, password): 
        try: 
            user = self.users[username] 
        except KeyError: 
            raise InvalidUsername(username) 
 
        if not user.check_password(password): 
            raise InvalidPassword(username, user) 
 
        user.is_logged_in = True 
        return True 
```

Notice how `KeyError` is handled. This could have been handled using `if username not in self.users:`  instead, but we chose to handle the exception directly. We end up  eating up this first exception and raising a brand new one of our own  that better suits the user-facing API.

We can also add a method to  check whether a particular username is logged in. Deciding whether to  use an exception here is trickier. Should we raise an exception if the username does not exist? Should we raise an exception if the user is not logged in?

To  answer these questions, we need to think about how the method would be  accessed. Most often, this method will be used to answer the yes/no  question, **should I allow them access to \<something>?** The answer will either be, **yes, the username is valid and they are logged in**, or **no, the username is not valid or they are not logged in**.  Therefore, a Boolean return value is sufficient. There is no need to  use exceptions here, just for the sake of using an exception:

```python
    def is_logged_in(self, username): 
        if username in self.users: 
            return self.users[username].is_logged_in 
        return False 
```

Finally, we can add a default authenticator instance to our module so that the client code can access it easily using `auth.authenticator`:

```python
authenticator = Authenticator() 
```

This line goes at the module level, outside any class definition, so the `authenticator` variable can be accessed as `auth.authenticator`. Now we can start on the `Authorizor` class, which maps permissions to users. The `Authorizor`  class should not permit user access to a permission if they are not  logged in, so they'll need a reference to a specific authenticator.  We'll also need to set up the permission dictionary upon initialization:

```python
class Authorizor: 
    def __init__(self, authenticator): 
        self.authenticator = authenticator 
        self.permissions = {} 
```

Now we can write methods to add new permissions and to set up which users are associated with each permission:

```python
    def add_permission(self, perm_name): 
        '''Create a new permission that users 
        can be added to''' 
        try: 
            perm_set = self.permissions[perm_name] 
        except KeyError: 
            self.permissions[perm_name] = set() 
        else: 
            raise PermissionError("Permission Exists") 
 
    def permit_user(self, perm_name, username): 
        '''Grant the given permission to the user''' 
        try: 
            perm_set = self.permissions[perm_name] 
        except KeyError: 
            raise PermissionError("Permission does not exist") 
        else: 
            if username not in self.authenticator.users: 
                raise InvalidUsername(username) 
            perm_set.add(username) 
```

The first  method allows us to create a new permission, unless it already exists,  in which case an exception is raised. The second allows us to add a  username to a permission, unless either the permission or the username  doesn't yet exist.

We use `set` instead of `list`  for usernames, so that even if you grant a user permission more than  once, the nature of sets means the user is only in the set once. We'll  discuss sets further in a later chapter.

A `PermissionError` error is raised in both methods. This new error doesn't require a username, so we'll make it extend `Exception` directly, instead of our custom `AuthException`:

```python
class PermissionError(Exception): 
    pass 
```

Finally, we can add a method to check whether a user has a specific `permission`  or not. In order for them to be granted access, they have to be both  logged into the authenticator and in the set of people who have been  granted access to that privilege. If either of these conditions is  unsatisfied, an exception is raised:

```python
    def check_permission(self, perm_name, username): 
        if not self.authenticator.is_logged_in(username): 
            raise NotLoggedInError(username) 
        try: 
            perm_set = self.permissions[perm_name] 
        except KeyError: 
            raise PermissionError("Permission does not exist") 
        else: 
            if username not in perm_set: 
                raise NotPermittedError(username) 
            else: 
                return True 
```

There are two new exceptions in here; they both take usernames, so we'll define them as subclasses of `AuthException`:

```python
class NotLoggedInError(AuthException): 
    pass 
 
class NotPermittedError(AuthException): 
    pass 
```

Finally, we can add a default `authorizor` to go with our default authenticator:

```python
authorizor = Authorizor(authenticator) 
```

That  completes a basic authentication/authorization system. We can test the  system at the Python prompt, checking to see whether a user, `joe`, is permitted to do tasks in the paint department:

```python
>>> import auth>>> auth.authenticator.add_user("joe", "joepassword")>>> auth.authorizor.add_permission("paint")>>> auth.authorizor.check_permission("paint", "joe")Traceback (most recent call last):  File "<stdin>", line 1, in <module>  File "auth.py", line 109, in check_permission    raise NotLoggedInError(username)auth.NotLoggedInError: joe>>> auth.authenticator.is_logged_in("joe")False>>> auth.authenticator.login("joe", "joepassword")True>>> auth.authorizor.check_permission("paint", "joe")Traceback (most recent call last):  File "<stdin>", line 1, in <module>  File "auth.py", line 116, in check_permission   raise NotPermittedError(username)auth.NotPermittedError: joe>>> auth.authorizor.check_permission("mix", "joe")Traceback (most recent call last):  File "auth.py", line 111, in check_permission    perm_set = self.permissions[perm_name]KeyError: 'mix'During handling of the above exception, another exception occurred:Traceback (most recent call last):  File "<stdin>", line 1, in <module>  File "auth.py", line 113, in check_permission    raise PermissionError("Permission does not exist")auth.PermissionError: Permission does not exist>>> auth.authorizor.permit_user("mix", "joe")Traceback (most recent call last):  File "auth.py", line 99, in permit_user    perm_set = self.permissions[perm_name]KeyError: 'mix'During handling of the above exception, another exception occurred:Traceback (most recent call last):  File "<stdin>", line 1, in <module>  File "auth.py", line 101, in permit_user    raise PermissionError("Permission does not exist")auth.PermissionError: Permission does not exist>>> auth.authorizor.permit_user("paint", "joe")>>> auth.authorizor.check_permission("paint", "joe")True
```

While verbose, the preceding output shows  all of our code and most of our exceptions in action, but to really  understand the API we've defined, we should write some exception  handling code that actually uses it. Here's a basic menu interface that  allows certain users to change or test a program:

```python
import auth

# Set up a test user and permission
auth.authenticator.add_user("joe", "joepassword")
auth.authorizor.add_permission("test program")
auth.authorizor.add_permission("change program")
auth.authorizor.permit_user("test program", "joe")


class Editor:
    def __init__(self):
        self.username = None
        self.menu_map = {
            "login": self.login,
            "test": self.test,
            "change": self.change,
            "quit": self.quit,
        }

    def login(self):
        logged_in = False
        while not logged_in:
            username = input("username: ")
            password = input("password: ")
            try:
                logged_in = auth.authenticator.login(username, password)
            except auth.InvalidUsername:
                print("Sorry, that username does not exist")
            except auth.InvalidPassword:
                print("Sorry, incorrect password")
            else:
                self.username = username

    def is_permitted(self, permission):
        try:
            auth.authorizor.check_permission(permission, self.username)
        except auth.NotLoggedInError as e:
            print("{} is not logged in".format(e.username))
            return False
        except auth.NotPermittedError as e:
            print("{} cannot {}".format(e.username, permission))
            return False
        else:
            return True

    def test(self):
        if self.is_permitted("test program"):
            print("Testing program now...")

    def change(self):
        if self.is_permitted("change program"):
            print("Changing program now...")

    def quit(self):
        raise SystemExit()

    def menu(self):
        try:
            answer = ""
            while True:
                print(
                    """
Please enter a command:
\tlogin\tLogin
\ttest\tTest the program
\tchange\tChange the program
\tquit\tQuit
"""
                )
                answer = input("enter a command: ").lower()
                try:
                    func = self.menu_map[answer]
                except KeyError:
                    print("{} is not a valid option".format(answer))
                else:
                    func()
        finally:
            print("Thank you for testing the auth module")


Editor().menu()
```

This rather long example is conceptually very simple. The `is_permitted` method is probably the most interesting; this is a mostly internal method that is called by both `test` and `change`  to ensure the user is permitted access before continuing. Of course,  those two methods are stubs, but we aren't writing an editor here; we're  illustrating the use of exceptions and exception handlers by testing an  authentication and authorization framework.