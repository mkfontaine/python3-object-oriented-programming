To wrap up this lesson, and the course, let's build a basic image  compression tool. It will take black and white images (with 1 bit per  pixel, either on or off) and attempt  to compress it using a very basic form of compression known as  run-length encoding. You may find black and white images a bit  far-fetched. If so, you haven't enjoyed enough hours [here](<http://xkcd.com>)!

I've included some sample black and white BMP images (which are easy to read data into and present plenty of opportunity to improve on file size) with the example code for this chapter.

Run-length  encoding takes a sequence of bits and replaces any strings of repeated  bits with the number of bits that are repeated. For example, the string  000011000 might be replaced with 04 12 03 to indicate that four zeros  are followed by two ones and then three more zeroes. To make things a  little more interesting, we will break each row into 127-bit chunks.

I  didn't pick 127 bits arbitrarily. 127 different values can be encoded  into 7 bits, which means that if a row contains all ones or all zeros,  we can store it in a single byte, with the first bit indicating whether  it is a row of 0s or a row of 1s, and the remaining seven bits  indicating how many of that bit exists.

Breaking up the image into  blocks has another advantage: we can process individual blocks in  parallel without them depending on each other. However, there's a major  disadvantage as well: if a run has just a few ones or zeros in it, then  it will take up `more` space in the compressed file. When we break up long runs into blocks, we may end up creating more of these small runs and bloat the size of the file.

We  have the freedom to design the layout of the bytes within the  compressed file as we see fit. For this simple example, our compressed  file will store two byte little-endian integers at the beginning of the  file representing the width and height of the completed file. Then, it  will write bytes representing the 127 bit chunks of each row.

Now, before we start designing a concurrent system to build such compressed images, we should ask a fundamental question: is this application I/O-bound or CPU-bound?

My answer, honestly, is **I don't know**.  I'm not sure whether the app will spend more time loading data from  disk and writing it back or doing the compression in memory. I suspect  that it is a CPU-bound app in principle, but once we start passing image  strings into subprocesses, we may lose any benefit of parallelism.

We'll  build this application using bottom-up design. That way, we'll have  some building blocks that we can combine into different concurrency  patterns to see how they compare. Let's start with the code that  compresses a 127-bit chunk using run-length encoding:

```python
from bitarray import bitarray 
def compress_chunk(chunk): 
    compressed = bytearray() 
    count = 1 
    last = chunk[0] 
    for bit in chunk[1:]: 
        if bit != last: 
            compressed.append(count | (128 * last)) 
            count = 0 
            last = bit 
        count += 1 
    compressed.append(count | (128 * last)) 
    return compressed
```

This code uses the `bitarray` class to manipulate individual zeros and ones. It is distributed as a third-party module, which you can install with the `pip install bitarray` command. The chunk that is passed into `compress_chunks`  is an instance of this class (although the example would work just as  well with a list of Booleans). The primary benefit of the `bitarray`  in this case is that, when pickling them between processes, they take  up an eighth of the space of a list of Booleans or a bytestring of 1s  and 0s. Therefore, they pickle faster. They are also a little easier to  work with than doing a ton of bitwise operations.

The method compresses the data using run-length encoding and returns `bytearray` containing the packed data. Where a `bitarray` is like a list of ones and zeros, `bytearray` is like a list of byte objects (each byte, of course, containing eight ones or zeros).

The  algorithm that performs the compression is pretty simple (although I'd  like to point out that it took me two days to implement and debug  it–simple to understand does not necessarily imply easy to write!). It first sets the `last` variable to the type of bit in the current run (either `True` or `False`).  It then loops over the bits, counting each one, until it finds one that  is different. When it does, it constructs a new byte by making the  leftmost bit of the byte (the 128 position) either a zero or a one,  depending on what the `last` variable  contained. Then, it resets the counter and repeats the operation. Once  the loop is done, it creates one last byte for the last run and returns  the result.

While we're creating building blocks, let's make a function that compresses a row of image data, as follows:

```python
def compress_row(row): 
    compressed = bytearray() 
    chunks = split_bits(row, 127) 
    for chunk in chunks: 
        compressed.extend(compress_chunk(chunk)) 
    return compressed
```

This function accepts a `bitarray` named `row`.  It splits it into chunks that are each 127 bits wide using a function  that we'll define very shortly. Then, it compresses each of those chunks  using the previously defined `compress_chunk`, concatenating the results into `bytearray`, which it returns.

We define `split_bits` as a generator, as follows:

```python
def split_bits(bits, width): 
    for i in range(0, len(bits), width): 
        yield bits[i:i+width]
```

Now, since we aren't  certain yet whether this will run more effectively in threads or  processes, let's wrap these functions in a method that runs everything  in a provided executor:

```python
def compress_in_executor(executor, bits, width): 
    row_compressors = [] 
    for row in split_bits(bits, width): 
        compressor = executor.submit(compress_row, row) 
        row_compressors.append(compressor) 
 
    compressed = bytearray() 
    for compressor in row_compressors: 
        compressed.extend(compressor.result()) 
    return compressed
```

This example barely needs explaining; it splits the incoming bits into rows based on the width of the image using the same `split_bits` function we have already defined (hooray for bottom-up design!).

Note that this code will compress any sequence of bits, although it would bloat, rather than compress  binary data that has frequent changes in bit values. Black and white  images are definitely good candidates for the compression algorithm in  question. Let's now create a function that loads an image file using the  third-party pillow module, converts it to bits, and compresses it. We  can easily switch between executors using the venerable comment  statement, as follows:

```python
from PIL import Image 
def compress_image(in_filename, out_filename, executor=None): 
    executor = executor if executor else ProcessPoolExecutor() 
    with Image.open(in_filename) as image: 
        bits = bitarray(image.convert('1').getdata()) 
        width, height = image.size 
 
    compressed = compress_in_executor(executor, bits, width) 
 
    with open(out_filename, 'wb') as file: 
        file.write(width.to_bytes(2, 'little')) 
        file.write(height.to_bytes(2, 'little')) 
        file.write(compressed) 
 
def single_image_main(): 
    in_filename, out_filename = sys.argv[1:3] 
    #executor = ThreadPoolExecutor(4) 
    executor = ProcessPoolExecutor() 
    compress_image(in_filename, out_filename, executor) 
```

The `image.convert()` call changes the image to black and white (one bit) mode, while `getdata()` returns an iterator over those values. We pack the results into a `bitarray`  so they transfer across the wire more quickly. When we output the  compressed file, we first write the width and height of the image  followed by the compressed data, which arrives as `bytearray`, which can be written directly to the binary file.

Having  written all this code, we are finally able to test whether thread pools  or process pools give us better performance. I created a large (7,200 x  5,600 pixels) black and white image and ran it through both pools. `ProcessPool` takes about 7.5 seconds to process the image on my system, while `ThreadPool` consistently takes about 9. Thus, as we suspected, the cost of pickling  bits and bytes back and forth between processes is eating almost all of  the efficiency gains from running on multiple processors (though,  looking at my CPU monitor, it does fully utilize all four cores on my  machine).

So, it looks like compressing a single image is most  effectively done in a separate process, but only barely, because we are  passing so much data back and forth between the parent and subprocesses.  Multiprocessing is more effective when the amount of data passed  between processes is quite low.

So, let's extend the app to  compress all the bitmaps in a directory in parallel. The only thing  we'll have to pass into the subprocesses are filenames, so we should get  a speed gain compared to using threads. Also, to be kind of crazy,  we'll use the existing code to compress individual images. This means  we'll be running `ProcessPoolExecutor` inside each subprocess to create even more subprocesses, as follows (I don't recommend doing this in real life!):

```python
from pathlib import Path 
def compress_dir(in_dir, out_dir): 
    if not out_dir.exists(): 
        out_dir.mkdir() 
 
    executor = ProcessPoolExecutor() 
    for file in ( 
            f for f in in_dir.iterdir() if f.suffix == '.bmp'): 
        out_file = (out_dir / file.name).with_suffix('.rle') 
        executor.submit( 
            compress_image, str(file), str(out_file)) 
 
def dir_images_main(): 
    in_dir, out_dir = (Path(p) for p in sys.argv[1:3]) 
    compress_dir(in_dir, out_dir) 
```

This code uses the `compress_image`  function we defined previously, but runs it in a separate process for  each image. It doesn't pass an executor into the function, so `compress_image` creates `ProcessPoolExecutor` once the new process has started running.

Now that we are running executors inside  executors, there are four combinations of threads and process pools  that we can be using to compress images. They each have quite different  timing profiles, as follows:

|                          | **Process pool per image** | **Thread pool per image** |
| ------------------------ | -------------------------- | ------------------------- |
| **Process pool per row** | 42 seconds                 | 53 seconds                |
| **Thread pool per row**  | 34 seconds                 | 64 seconds                |

As we might expect, using threads for each image and again using threads for each row is the slowest configuration,  since the GIL prevents us from doing any work in parallel. Given that  we were slightly faster when using separate processes for each row when  we were using a single image, you may be surprised to see that it is  faster to use a `ThreadPool` feature for rows if we are processing each image in a separate process. Take some time to understand why this might be.

My  machine contains only four processor cores. Each row in each image is  being processed in a separate pool, which means that all those rows are  competing for processing power. When there is only one image, we get a  (very modest) speedup by running each row in parallel. However, when we  increase the number of images being processed at once, the cost of  passing all that row data into and out of a subprocess is actively  stealing processing time from each of the other images. So, if we can  process each image on a separate processor, where the only thing that  has to get pickled into the subprocess pipe is a couple of filenames, we  get a solid speedup.

Thus, we see that different workloads  require different concurrency paradigms. Even if we are just using  futures, we have to make informed decisions about what kind of executor  to use.

Also note that for typically-sized images, the program  runs quickly enough that it really doesn't matter which concurrency  structures we use. In fact, even if we didn't use any concurrency at  all, we'd probably end up with about the same user experience.

This  problem could also have been solved using the threading and/or  multiprocessing modules directly, though there would have been quite a  bit more boilerplate code to write. You may be wondering whether or not AsyncIO would be useful here. The answer is: **probably not**. Most operating systems don't have a good way to perform non-blocking reads from the filesystem, so the library ends up wrapping all the calls in futures anyway.

For completeness, here's the code that I used to decompress the **run-length encoding** (**RLE**)  images to confirm that the algorithm was working correctly (indeed, it  wasn't until I fixed bugs in both compression and decompression, and I'm  still not sure if it is perfect–I should have used test-driven  development!):

```python
from PIL import Image 
import sys 
 
def decompress(width, height, bytes): 
    image = Image.new('1', (width, height)) 
 
    col = 0 
    row = 0 
    for byte in bytes: 
        color = (byte & 128) >> 7 
        count = byte & ~128 
        for i in range(count): 
            image.putpixel((row, col), color) 
            row += 1 
        if not row % width: 
            col += 1 
            row = 0 
    return image 
 
 
with open(sys.argv[1], 'rb') as file: 
    width = int.from_bytes(file.read(2), 'little') 
    height = int.from_bytes(file.read(2), 'little') 
 
    image = decompress(width, height, file.read()) 
    image.save(sys.argv[2], 'bmp') 
```

This code is  fairly straightforward. Each run is encoded in a single byte. It uses  some bitwise math to extract the color of the pixel and the length of  the run. Then, it sets each pixel from that run in the image,  incrementing the row and column of the next pixel to analyze at appropriate intervals.