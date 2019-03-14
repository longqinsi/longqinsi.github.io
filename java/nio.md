# [技术备忘录](../README.md) | [Java](README.md) | NIO/NIO2
## 目录
  1. [NIO的全称是什么？](#what-does-nio-stands-for)
  2. [What does nio bring to Java?](#nio-features)
  3. [What does nio2 bring to Java?](#nio2-features)
  4. [JDK中用于访问文件和网络的API有哪些？](#java-io-api-versions)

## 问题
### 1.NIO的全称是什么？<a name="what-does-nio-stands-for"></a>[↑](#top)
Non-blocking Input Output

### 2.What does nio bring to Java?<a name="nio-features"></a>[↑](#top)
* Support for very large buffers.
* Asynchronous capabilities.

### 3.What does nio2 bring to Java?<a name="nio2-features"></a>[↑](#top)
* Native access to file systems.
* API to explore very large directory trees.
* Respond to file creations and events.

Reference source: Questions 2 & 3 are cited from José Paumard's pluralsight course
[Java Fundamentals: NIO and NIO2](https://app.pluralsight.com/library/courses/java-fundamentals-nio-nio2)

### 4.JDK中用于访问文件和网络的API有哪些？<a name="java-io-api-versions"></a>[↑](#top)

名称 | 发布年份 | JDK版本
------ | ------ | -------
Java I/O | 1996 | Java 1
Java NIO | 2002 | Java 4
Java NIO 2 | 2011 | Java 7

### 5.What new concepts has Java NIO introduced？<a name="nio-new-concepts"></a>[↑](#top)
Buffers, Channels, and Selectors.

#### Buffer: where the data resides.
> A buffer can be ssen as a space in memory. It can reside in heap or off-heap(which is useful for very large buffers).

#### Channel: where the data comes from.
> The [channel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/Channel.html) object is an object that connects to a file or to a socket for instance. The channel can write the buffer to a medium or can read data from that medim to a buffer. A channel only knows bytes buffers so it can only read and write bytes from files or sockets. Afterwards, we will have to convert the content of this buffer to characters if this is a character buffer or to data or object if it is raw data or raw objects.

> Channels 包括：file, socket, in-memory.

> A write operation takes data from a buffer and writes it to a channel.

This is how we can write data to a file, for instance. There are ways of filling buffers with:
   * characters
   * raw bytes
   * primitive data types
   * java objects

> A read operation reads data from a channel and writes it into a buffer.

Once the data is in the buffer we can read it and interpret it as:
   * characters
   * raw bytes
   * primitive data types
   * java objects

#### Selector: handle asynchronous operations.



José Paumard [Java Fundamentals: NIO and NIO2](https://app.pluralsight.com/library/courses/java-fundamentals-nio-nio2):
> NIO stands for non-blocking IO. It deals with buffers and channels, and supports asynchronous operations.
> NIO **may** be more efficient than I/O, but it really depends on your application and your use case.

### 6.对NIO2的概论<a name="nio2-introduction"></a>[↑](#top)

José Paumard [Java Fundamentals: NIO and NIO2](https://app.pluralsight.com/library/courses/java-fundamentals-nio-nio2):
>NIO2 brought some more functionalities to both Java I/O and Java NIO:
> 1. Native access to file systems.
> 2. Events and specialy directory events to track file creations, deletions and modifications.
> 3. A very powerful directory structure exploration API

### 7. Why Java NIO?
The shortcomings of Java I/O
1. Java I/O writes/reads one byte/char at a time, and cannot conduct bulk reads or bulk write operation.
2. Readers are for reading only, and writers are for writing only.
3. Buffering ([BufferedInputStream](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedInputStream.html), [BufferedOutputStream](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedOutputStream.html), [BufferedReader](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedReader.html), and [BufferedWriter](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedWriter.html)) occurs in the JVM heap memory, which is not well adapted to very large files, very large read and write operations.
4. The handling of charsets is not great. i.e. a Lating-1 encoded file, that is a charset not supported by default by the JDK which defaults UTF-8, would have to be read using the binary operation and then converted character by character to the right charset. And this is not completely transparent to the user.
5. All the operations are synchronous.

All this was not that bad in 1995, but in 2002, seven years later, people had the feeling that those functionalities were missing from Java I/O and decided to create a new Java I/O API to handle all this. This is why NIO was built.

NIO provides:
1. Bulk access to raw bytes.
2. Bidrectional channels.
   * See package [java.nio.channels](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/package-frame.html)
3. Off-heap buffering.
   * See package [java.nio](https://docs.oracle.com/javase/8/docs/api/java/nio/package-frame.html)
   * You can create buffer outside the central memory of the JVM in portions of memory not handled by the garbage collector. So you can create very large buffers, in GBs, even in TBs without any impact on the performance of the garbage collector.
4. Proper support for charsets.
   * See package [java.nio.charset](https://docs.oracle.com/javase/8/docs/api/java/nio/charset/package-frame.html)
   * JDK defines standard [Charset](https://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html)s objects - objects for the standard, well-known, and most widely used charset and those charsets provide [encode](https://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#encode-java.nio.CharBuffer-) and [decode](https://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#decode-java.nio.ByteBuffer-) methods to convert a stream expressed in a given charset to another charset.
5. Support for asynchronous operations.

### 8.Understanding Channels?
[Channel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/Channel.html) is an interface, implemented by:
  1. [FileChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/FileChannel.html) accesses to a file.
      - It has a cursor
      - It allows for multiple reads and writes
      - It is thread safe
  2. [DatagramChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/DatagramChannel.html) accesses to a UDP socket.
      - It supports multicast since it is **UDP**.
      - It supports multiple **non-concurrent** reads and writes.
  3. [SocketChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SocketChannel.html) and [ServerSocketChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SocketChannel.html) access to TCP sockets.
      - It supports asynchronous operations.
      - It also supports multiple **non-concurrent** reads and writes.

FileChannel, SocketChannel and ServerSocketChannel are **abstract** classes. Concrete implementations are hidden and should not be used directly.

Channels are created with **factory** methods.

Channel factory method reference list:
   - [FileChannel.open(Path path, OpenOption... options)](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/FileChannel.html#open-java.nio.file.Path-java.nio.file.OpenOption...-)
     - Opens or creates a file, returning a file channel to access the file.
   - [FileChannel.open(Path path, Set<? extends OpenOption> options, FileAttribute<?>... attrs)](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/FileChannel.html#open-java.nio.file.Path-java.util.Set-java.nio.file.attribute.FileAttribute...-)
     - Opens or creates a file, returning a file channel to access the file.
   - [SocketChannel.open()](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SocketChannel.html#open--)
     - Opens a socket channel and connects it to a remote address.
   - [SocketChannel.open(SocketAddress remote)](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SocketChannel.html#open-java.net.SocketAddress-)
     - Opens a socket channel and connects it to a remote address.
   - [ServerSocketChannel.open()](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/ServerSocketChannel.html#open--)
     - Opens a server-socket channel.

### 9. FileChannel dicrectly mapping to memory array.
* A FileChannel can be mapped to a memory array for direct access. This allows for much faster operation than accessing to the disk. It is built on native features provided by different operation systems and this is why the concrete implementations of FileChannel are hidden just because they are dependent on the machine we are working on. 
* It should be used with caution because a single write in this array can trigger a modification of the file that will be sent to disk directly.<a name="filechannel-mapping-warning"></a>
* There are 3 modes for this mapping:
  * READ_ONLY: The file is just loaded into memory and is read from this array. The mapped file cannot be modified.
  * READ_WRITE: The file can be modified with [the previous waring](#filechannel-mapping-warning)
  * PRIVATE: modifications are **local** to this channel and will not be propagated to the disk.

### 10. Buffers
* Buffer is an abstract class, extended by typed buffers.
  * [ByteBuffer](https://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html)
    * It is the only type of buffer a channel can write in or read from.
  * [CharBuffer](https://docs.oracle.com/javase/8/docs/api/java/nio/CharBuffer.html)
    * If we need to write characters to a ByteBuffer we need to decorate it with a CharBuffer.
  * There are [IntBuffer](https://docs.oracle.com/javase/8/docs/api/java/nio/IntBuffer.html), [DoubleBuffer](https://docs.oracle.com/javase/8/docs/api/java/nio/DoubleBuffer.html), [LongBuffer](https://docs.oracle.com/javase/8/docs/api/java/nio/LongBuffer.html), [FloatBuffer](https://docs.oracle.com/javase/8/docs/api/java/nio/FloatBuffer.html), [ShortBuffer](https://docs.oracle.com/javase/8/docs/api/java/nio/ShortBuffer.html) that allow for the reading and writing of int, double, long, float, short into a ByteBuffer respectively.
  * These buffers(ByteBuffer, CharBuffer, etc.) are extended by concrete implementations. The concrete implementations are hidden and we do not have direct access to them. We can only create buffer instances using **factory** methods.
* A buffer is an in-memory structure, backed by an array of bytes. It is usually stored in the central memory of the JVM, handled by the garbage collector. But it can also be stored in the off-heap space of the JVM, thus not impacting the garbage collector. And this is very useful for very large buffers. The size of a buffer is an int since it is backed by an array. So the size of a buffer can be as large as 2 GB which could have an impact on the performance of the garbage collector if stored in the central memory space of the JVM. 
* A buffer object has 3 properties:
  * a capacity = the size of the backing array
  * a current position = cursor
    * All the read and the write operation are made to or from the current position.
  * a limit = the last position in memory seen by this buffer
    * With cursor and limit, we can create views on buffers which can seen as a kind of a sub buffer inside a buffer.
* A buffer always keeps track of the available space.
* **Mark**
  * A buffer can hold a single mark. [Marking](https://docs.oracle.com/javase/8/docs/api/java/nio/Buffer.html#mark--) a buffer does two things:
    1. It sets the mark to the current position.
    2. It returns ```this``` to be able to chain calls on this buffer.
* A buffer supports 4 operatons:
  1. [rewind](https://docs.oracle.com/javase/8/docs/api/java/nio/Buffer.html#rewind--)
     * Clears the mark and set the current position to 0
  2. [reset](https://docs.oracle.com/javase/8/docs/api/java/nio/Buffer.html#reset--)
     * Sets the current position to the previously set mark
  3. [flip](https://docs.oracle.com/javase/8/docs/api/java/nio/Buffer.html#flip--)
     * Sets the limit to the current position and rewinds the buffer
  4. [clear](https://docs.oracle.com/javase/8/docs/api/java/nio/Buffer.html#clear--)
     * Clears the buffer
  
  All these operations along with the mark operation return ```this``` and can be chained.

### 11. How to write content to a file using buffers and channels?
  A channel can only write to a ByteBuffer.
  1. create a ByteBuffer.
  2. use the putXXX() method to write data
  3. have our file channel to write the ByteBuffer to a file
  
  ```java
  ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
  byteBuffer.putInt(10);
  byteBuffer.rewind();
  try(FileChannel fileChannel = FileChannel.open(
    Paths.get("files/ints.bin"), StandardOpenOption.CREATE, StandardOpenOption.WRITE)){
      fileChannel.write(byteBuffer);
    }
  ```

### 12. How to read a file using nio?
  * In order to properly read a file we need to understand how buffers work.
    * First, we create a channel on the given file
    * Then read the content of the file into the byte buffer
    * Then read the byte buffer and translate its content
  ![在使用channel把数据读取到byte buffer后，调用byte buffer的asXXXBuffer()方法之前，需要调用其flip方法设置position和limit](/images/java-nio-buffer-flip.png)

### 13. Rewind vs. Flip
* Rewind just resets the cursor.
* Flip resets the cursor and prevents readings past what has been written into the buffer.
  
So to properly conduct read operation, most of the time it is the flip operation that it used.

### 14. Reading and writing to multiple buffers at the same time
* The reading of a file into multiple buffers is called **the scattering read operation**. 
It consists reading from a single channel, that is from a single file, to an array of
buffers. 
  * It is specified by the [ScatteringByteChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/ScatteringByteChannel.html) interface.
  * The reading process fills the first buffer before moving to the next, and continues with the
next one and so on.
* The opposite is called **the gathering write operation**, which consists writing from an array
of buffers to a single Channel.
  * It is specified by the [GatheringByteChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/GatheringByteChannel.html).
  * Writing operation does the exact opposite as the reeading operation. It starts with the first
    buffer, then the next one and so on.
* The **Gather/Scatter** pattern is useful when handling messages with **fixed-length parts**.
  * Suppose we have a 1024 bytes header followed by a 4096 bytes body, ending with a 128 bytes footer.
    Then we can set up an array of three buffers of the right size, plug a channel on these buffers,
    then the first reading will fill the first buffer, second reading will fill the body, and
    the third reading will fill the footer.
    ![](/images/java-nio-fixed-length-messages.png)
* The read operation example:
```java
ByteBuffer header = ByteBuffer.allocate(1024);
ByteBuffer body   = ByteBuffer.allocate(4096);
ByteBuffer footer = ByteBuffer.allocate(128);

ByteBuffer[] message = {header, body, footer};

long bytesRead = channel.read(message);
```
* The write operation example:
```java
ByteBuffer header = ByteBuffer.allocate(1024);
ByteBuffer body   = ByteBuffer.allocate(4096);
ByteBuffer footer = ByteBuffer.allocate(128);

ByteBuffer[] message = {header, body, footer};

long bytesRead = channel.read(message);
```
**Remember to properly rewind the buffers when using them.**
### 15. [MappedByteBuffer](https://docs.oracle.com/javase/8/docs/api/java/nio/MappedByteBuffer.html)
MappedByteBuffer is a buffer that maps a file to memory.

Think of a buffer that is able to load a file in memory thus all the parts of
your application that are reading the same file again and again will be much
more efficient since the readings will take place in memory instead of taking
place on the disk.

There are three modes for MappedByteBuffers:
  * READ - probably the most useful one
  * READ_WRITE - you can both read and modify a file through the buffer
  * PRIVATE - The modifications will be made private

The way the MappedByteBuffer is created also allows for the buffering of a **portion**
of a **file** instead of the whole file itself.
```java
FileChannel fileChannel = FileChannel.open(
  Paths.get("files/ints.bin"), StandardOpenOption.READ);
MappedByteBuffer mappedBuffer = fileChannel.map(
  FileChannel.MapMode.READ_ONLY, 0, fileChannel.size());
CharBuffer charBuffer = StandardCharsets.UTF_8.decode(mappedBuffer);
```
### 16. Using Charsets to convert byte buffers to char buffers.
A charset has two methods:
- [encode()](https://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#encode-java.nio.CharBuffer-): takes a CharBuffer, returns a ByteBuffer
- [decode()](https://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#decode-java.nio.ByteBuffer-): takes a ByteBuffer, returns a CharBuffer

Using these methods is the **only way** to convert a **CharBuffer** to a **ByteBuffer** and vice-versa.
```java
FileChannel channel = FileChannel.open(
  Paths.get("files/text-latin1.txt", StandardOpenOption.READ));
ByteBuffer buffer = ByteBuffer.allocate(1024);
channel.read(buffer);

CharSet latin1 = StandardCharsets.ISO_8859_1;
CharBuffer utf8Buffer = latin1.decode(buffer);

String result = utf8Buffer.toString();
System.out.println("Result = " + result);
```

```java
CharSet utf8 = StandardCharsets.UTF_8;
ByteBuffer byteBuffer = utf8.encode(buffer);

anotherFileChannel.write(byteBuffer);
```

### 17. Buffers and Charsets
* Channels can only read and write byte buffers
* Encoding and decoding via a Charset is the only way to convert a byte buffer to a char buffer and vice-versa.

### 18. [Channels](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/Channels.html) - bridge between Java NIO and Java I/O API
* to create channels from InputStream and OutputStream
* to create InputStream and OutputStream from a channel, asynchronous or not
* to create readers and writers from a channel, providing a charset
