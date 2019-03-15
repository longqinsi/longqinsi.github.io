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

### 4. Code sample to get file systems and stores from the FileSystemProvider

```java
List<FileSystemProvider> providers = FileSystemProvider.installedProviders();
FileSystem defaultFileSystem = providers.get(0).getFileSystem(URI.create("file:///"));
// We can also get the default(disk) file system with the factory class
FileSystem defaultFileSystem1 = FileSystems.getDefault(); 
// Or by providing the right URI
FileSystem defaultFileSystem2 = FileSystems.getFileSystem(URI.create("file:///")); 
// Then we can get the root directories of this file system
// File.listRoots() is equivalent to fileSystem.getRootDirectories(),
// but File.listRoots() returns an array, which can be very costly, especially if we
// have many, many root directories on our file system. And this is how NIO2 has been
// designed. Every time we need to get the set of elements, for instance here, the root
// directories of our file system, but think also of the files available in a given
// direcotry, it always an Iterable. Why? Because an Iterable is a lazy structure. It
// does not hold the result, so it is much more efficient than returning a list.
Iterable<Path> roots = defaultFileSystem.getRootDirectories();
// We can also get the file stores from this default file system, and it returns an
// Iterable of FileStore objects, which contains more information.
Iterable<FileStore> stores = defaultFileSystem.getFileStores();
// And from this FileStore, we can get information that were not available through
// the use of the plain Java I/O API. We can get the name and the type of the store.
FileStore store = stores.iterator().next();
store.name(); // Data_1
store.type(); // NTFS
```

### 5. How to create I/O and NIO objects with the FileSystem object?
FileSystemProvider is the entry point:
* Java I/O Operations:
  * FileSystemProvider.newInputStream(Path, OpenOption)
  * FileSystemProvider.newOutputStream(Path, OpenOption)
* Java NIO Operations:
  * FileSystemProvider.newFileChannel(Path, ...)
  * FileSystemProvider.newByteChannel(Path, ...)
  * FileSystemProvider.newAsynchronousFileChannel(Path, ...)

### 6. How to create directories with NIO2?
Using Files or FileSystemProvider:
* fileSystemProvider.createDirectory(path, attributes)
```java
Path path = Paths.get("C:/tmp");
FileSystemProvider defaultProvider = FileSystemProvider.installedProviders().get(0);
defaultProvider.createDirectory(path);
```
* Files.createDirectory(path, attributes)

### 7. A Path is Bound to a File System
* If the path is created from a String, then it is bound to the default file system(that is the disk).
  * [Paths.get(String first, String... more)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Paths.html#get(java.lang.String,java.lang.String...))
* If the path is created from a URI, then it is bound to the file system with the corresponding scheme
  * [Paths.get(URI uri)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Paths.html#get(java.net.URI))

### 8. Accessing File Attributes

Java NIO2 gives access to file attributes both for Windows and Unix file systems.
Reading the attributes of a file is done through the file system provider ([FileSystemProvider](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/spi/FileSystemProvider.html)) or [Files](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html)
Three interfaces are involved:
1. [BasicFileAttributes](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/attribute/BasicFileAttributes.html) - common to all the file systems
2. [DosFileAttributes](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/attribute/DosFileAttributes.html)   - with methods specific to the Windows file systems
3. [PosixFileAttributes](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/attribute/PosixFileAttributes.html) - with methods to get Posix-specific file attributes
![Java NIO2中用于读取文件属性的三个接口](/images/java-nio2-file-attributes-interfaces.png)

```java
Path path = Paths.get("C:/tmp/file.txt");
// First, use FileSystemProviders to get the file attributes object
DosFileAttributes attributes = path.getFileSystem().provider().readAttributes(path, DosFileAttributes.class);
// Second, use Files to get it
attributes = Files.readAttributes(path, DosFileAttributes.class);
```

### 9. Jar File System
There are two modes:
1. creation of ZIP files
2. reading of ZIP files

A ZIP file can be written or read in two ways:

1. by copying existing files in it
2. by writing content directly in it

```java
URI zipFile = URI.create("jar:file:///C:/tmp/archive.zip");
Map<String, String> options = new HashMap<>();
options.put("create", "true");
options.put("encoding", "UTF-8");

FileSystem zipFS = FileSystems.newFileSystem(zipFile, options);

Path someText = Paths.get("files/some.txt");
Files.copy(someText, zipFS.getPath("some.txt"));

Path dir = zipFS.getPath("files");
// Another way: create path through a URI
dir = Paths.get(URI.create("jar:file:///C:/tmp/archive.zip!/files"));

Files.createDirectory(dir);
Files.copy(someText, dir.resolve("some.txt"));
```