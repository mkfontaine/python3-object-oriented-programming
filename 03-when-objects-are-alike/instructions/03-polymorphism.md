We were introduced to polymorphism in **Object-Oriented Design**. It is a showy name describing a simple concept: different behaviors  happen depending on which subclass is being used, without having to  explicitly know what the subclass actually is. As an example, imagine a  program that plays audio files. A media player might need to load an `AudioFile` object and then `play` it. We can put a `play()`  method on the object, which is responsible for decompressing or  extracting the audio and routing it to the sound card and speakers. The  act of playing an `AudioFile` could feasibly be as simple as:

```python
audio_file.play() 
```

However, the process of decompressing and extracting an audio file is very different for different types of files. While `.wav` files are stored uncompressed, `.mp3`, `.wma`, and `.ogg` files all utilize totally different compression algorithms.

We  can use inheritance with polymorphism to simplify the design. Each type  of file can be represented by a different subclass of `AudioFile`, for example, `WavFile` and `MP3File`. Each of these would have a `play()`  method that would be implemented differently for each file to ensure  that the correct extraction procedure is followed. The media player  object would never need to know which subclass of `AudioFile` it is referring to; it just calls `play()`  and polymorphically lets the object take care of the actual details of  playing. Let's look at a quick skeleton showing how this might look:

```python
class AudioFile:
    def __init__(self, filename):
        if not filename.endswith(self.ext):
            raise Exception("Invalid file format")

        self.filename = filename


class MP3File(AudioFile):
    ext = "mp3"

    def play(self):
        print("playing {} as mp3".format(self.filename))


class WavFile(AudioFile):
    ext = "wav"

    def play(self):
        print("playing {} as wav".format(self.filename))


class OggFile(AudioFile):
    ext = "ogg"

    def play(self):
        print("playing {} as ogg".format(self.filename))
```

All audio files check to ensure that a valid extension was given upon initialization. But did you notice how the `__init__` method in the parent class is able to access the `ext`  class variable from different subclasses? That's polymorphism at work.  If the filename doesn't end with the correct name, it raises an  exception (exceptions will be covered in detail in the next chapter).  The fact that the `AudioFile` parent class doesn't actually store a reference to the `ext` variable doesn't stop it from being able to access it on the subclass.

In addition, each subclass of `AudioFile` implements `play()`  in a different way (this example doesn't actually play the music; audio  compression algorithms really deserve a separate book!). This is also  polymorphism in action. The media player can use the exact same code to  play a file, no matter what type it is; it doesn't care what subclass of  `AudioFile` it is looking at. The details of decompressing the audio file are **encapsulated**. If we test this example, it works as we would hope:

```python
>>> ogg = OggFile("myfile.ogg")>>> ogg.play()playing myfile.ogg as ogg>>> mp3 = MP3File("myfile.mp3")>>> mp3.play()playing myfile.mp3 as mp3>>> not_an_mp3 = MP3File("myfile.ogg")Traceback (most recent call last):  File "<stdin>", line 1, in <module>  File "polymorphic_audio.py", line 4, in __init__    raise Exception("Invalid file format")Exception: Invalid file format
```

See how `AudioFile.__init__` is able to check the file type without actually knowing which subclass it is referring to?

Polymorphism  is actually one of the coolest things about object-oriented  programming, and it makes some programming designs obvious that weren't  possible in earlier paradigms. However, Python makes polymorphism seem  less awesome because of duck typing. Duck typing in Python allows us to  use **any** object that provides the  required behavior without forcing it to be a subclass. The dynamic  nature of Python makes this trivial. The following example does not  extend `AudioFile`, but it can be interacted with in Python using the exact same interface:

```python
class FlacFile: 
    def __init__(self, filename): 
        if not filename.endswith(".flac"): 
            raise Exception("Invalid file format") 
 
        self.filename = filename 
 
    def play(self): 
        print("playing {} as flac".format(self.filename)) 
```

Our media player can play this object just as easily as one that extends `AudioFile`.

Polymorphism  is one of the most important reasons to use inheritance in many  object-oriented contexts. Because any objects that supply the correct  interface can be used interchangeably in Python, it reduces the need for  polymorphic common superclasses. Inheritance can still be useful for  sharing code, but if all that is being shared is the public interface,  duck typing is all that is required. This reduced need for inheritance  also reduces the need for multiple inheritance; often, when multiple  inheritance appears to be a valid solution, we can just use duck typing  to mimic one of the multiple superclasses.

Of course, just because  an object satisfies a particular interface (by providing required  methods or attributes) does not mean it will simply work in all  situations. It has to fulfill that interface in a way that makes sense  in the overall system. Just because an object provides a `play()` method does not mean it will automatically work with a media player. For example, our chess AI object from **Object-Oriented Design**, may have a `play()`  method that moves a chess piece. Even though it satisfies the  interface, this class would likely break in spectacular ways if we tried  to plug it into a media player!

Another useful feature of duck typing  is that the duck-typed object only needs to provide those methods and  attributes that are actually being accessed. For example, if we needed  to create a fake file object to read data from, we can create a new  object that has a `read()` method; we don't have to override the `write`  method if the code that is going to interact with the fake object will  not be calling it. More succinctly, duck typing doesn't need to provide  the entire interface of an object that is available; it only needs to  fulfill the interface that is actually accessed.