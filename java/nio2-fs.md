# [技术备忘录](../README.md) | [Java](README.md) | NIO2与文件系统
## 目录
  1. [NIO2提供了哪些类用于访问文件系统？](#nio2-fs-classes)

## 问题
### 1. NIO2提供了哪些类用于访问文件系统？<a name="nio2-fs-classes"></a>[↑](#top)
* NIO2 API provides classes to access the file system directly, and implements file systems dependent operations not provided by Java I/O:
  * The [FileSystemProvider](https://docs.oracle.com/javase/8/docs/api/java/nio/file/spi/FileSystemProvider.html) class
    * FileSystemProvider acts as a factory for file systems
      * can create other file systems.
      * provides the mothods to create/move/copy/delete files, links and directories(the mothods for links and directories are new in the Java Input/Output space).
      * works with Java I/O and Java NIO, providing bridges for objects from those two APIs.
      * gives access to security attributes
        * special attributes particular to a special file system. For example, a Windows file system and a Linux file system do not have the same security attributes, through the file system provider, we can get them in a native way, and this was not possible in Java I/O.
  * The [FileSystem](https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileStore.html) class
    * FileSystem is an abstraction of a file system
      * can close the file system
      * can query if it is opened
      * can query if it is read only
      * provides technical information: the root directories, the used separator.
      * can get the stores as FileStore objects
      * can create a path in that file system
        * **This is a very important point to note.**
        * [Path](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Path.html) is an interface, and the implementation of this interface is dependent on the file system it is created from.
      * can create a watch service

  * The [FileStore](https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileStore.html) class
    * FileStore is an abstraction of a file store within a file system
      * provides the **name** and **type** of the store
        * These two informations were not available through the file class of the Java I/O
      * provides information on the **space** of the store
        * the used space, the available space, etc.
      * gives access to **security attributes** in a native way
  * The [FileSystems](https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileSystems.html) class
    * A factory class to create file systems, provides facade methods to the FileSystemProvider class.

### 2. Why add file system support in NIO2?

* Now that we already have methods to move files around, to create directories,
to create files, to check for the existence of files and direcotries, and we
can get the content of a directory. We have methods for that in the [File](https://docs.oracle.com/javase/8/docs/api/java/io/File.html) class.

* The answer is: **performance**
  
* In fact, the mothods we have from Java I/O are okay, they work; but when it
comes to handling very large directories with many, many files in them,
think about thousands of files, it is becoming less and less efficient.

* So, this new API from NIO2 is directly plugged into the native file system
and can handle very large directories even in its java implementation.

* So, the main reasons are：
  1. **performance**
  2. **get more functionalities from the native file system**

### 3. What is a File System?
A file system in Java NIO2 is an abstraction of a real file system. It is a new concept in the JDK; did not exist before JDK 7. 

It is bound to a scheme in the URI sense. And the default scheme is file://. 

It implements all the operations of the [Files](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html) factory class (creation, copy, deletion, etc...).

By default, the JDK provides two file systems:

  1. The default file system which is the classical disk file system
  2. A JAR file system that can be set up in memory or directly on the disk.

### 4. 在Java 7中引入了什么机制来监控文件系统的变化？
[WatchService](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/WatchService.html) pattern.

It still uses a special thread
But no scheduler
No events are missed(almost)
  * Some events may be missed in some cases.
  * It is plugged directly on the signals emitted by the native file system.

Setting up a watch service is a four-steps process:
1. create a watch service
2. register the watch service to a directory
3. gets the returned key
4. poll the events and analyze them.

```java
Path dir = Paths.get("D:/logs");
FileSystem fileSystem = dir.getFileSystem();
WatchService watchService = fileSystem.newWatchService();
WatchKey key = dir.register(watchService,
                       StandardWatchEventKinds.ENTRY_CREATE,
                       StandardWatchEventKinds.ENTRY_DELETE,
                       StandardWatchEventKinds.ENTRY_MODIFY);
while (key.isValid()) {
  // or poll with / without a timeout
  WatchKey take = watchService.take();
  List<WatchEvent<?>> events = take.pollEvents();
  // work with the events
  for (WatchEvent<?> event : events) {
    WatchEvent.Kind<?> kind = event.kind();
    if(kind == StandardWatchEventKinds.OVERFLOW) { // overflow
      continue;
    }
    // operations on the elements
  }
  take.reset();
}
```

* The WatchKey Object
  * [isValid()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/WatchKey.html#isValid()) returns true as long as the directory is accessible
  * Three methods to poll events:
    * [take()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/WatchService.html#take()): a blocking call
      * as long as there are no events availabe, this method will not return
    * [poll()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/WatchService.html#poll()): non-blocking, can return null
    * [poll(long, TimeUnit)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/WatchService.html#poll(long,java.util.concurrent.TimeUnit)): poll with a timeout, can return null

* If there are too many events generated, an OVERFLOW is added to the queue.
Some events may have been missed.