# [技术备忘录](../README.md) | [Java](README.md) | NIO2与目录树
## 目录
  1. [为什么NIO2提供了新的API用于遍历目录？](#why-nio2-new-dir-apis)

## 问题
### 1. 为什么NIO2在Java I/O已经有用于遍历目录的API的情况下，又增加了新的、具有类似功能API？<a name="why-nio2-new-dir-apis"></a>[↑](#top)
为了效率，新的API比Java I/O的旧API性能更好，因为它针对特定的操作系统、文件系统而实现。

### 2. NIO2提供了哪些API用于遍历目录？<a name="nio2-dir-apis"></a>[↑](#top)
* Path matchers
  * used to filter paths from a directory tree

### 3. Directory Streams and Matchers

A **directory stream** is a way of analyzing the content of a directory.

It doesn't explore the subdirectories but those directories are still part of the analysis.

It can be used to get all the content of a directory, and it can also filter its content by
providing a lambda expression.

```java
Path dir = Paths.get("C:/files");

// use lambda expression or method reference as a filter to get a directory stream
DirectoryStream<Path> directoryStream1 = Files.newDirectoryStream(dir, Files::isDirectory);

// use regular expression to match file & directory names
DirectoryStream<Path> directoryStream2 = Files.newDirectoryStream(dir, "*.java");

// If we need complex file name checking, we can use a file matcher
PathMatcher pathMatcher = FileSystems.getDefault().getPathMatcher("glob:**/*.java");
DirectoryStream<Path> directoryStream3 = Files.newDirectoryStream(dir, pathMatcher::matches);

```

The path matcher allows for two kinds of regular expressions:
  * regex: specified in the [Pattern](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Pattern.html) class
  * glob: which is a simplified version of regex:, specified in the [FileSystem.getPathMatcher](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/FileSystem.html#getPathMatcher(java.lang.String)) method.

```java
DirectoryStream<Path> directoryStream = ...;
for (Path path: directoryStream) {
  // operations on the elements
}
```
```java
DirectoryStream<Path> directoryStream = ...;
directoryStream.forEach(System.out::println);
```
```java
DirectoryStream<Path> directoryStream = ...;
List<Path> paths = 
  StreamSupport.stream(directoryStream.spliterator(), false)
    .collect(Collectors.toList());
```

### 4. Walking Directory Trees Using [Files.walk](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#walk(java.nio.file.Path,java.nio.file.FileVisitOption...))
Walking a directory tree consists in exploring all the files and subdirectories.

Java uses the **depth-first** approach to do it.

* The Files.walk methods walk a directory tree.
  * The parameters are:
    1. the starting point as a path (it should be a directory)
    2. the maximum depth to be explored (optional)
    3. an option to decide to follow symbolic links or not (optional, by default not follows)

```java
Path dir = Paths.get("D:/sources");
Stream<Path> paths1 = Files.walk(dir);
Stream<Path> paths2 = Files.walk(dir, 3);
Stream<Path> paths3 = Files.walk(dir, 3, FileVisitOption.FOLLOW_LINKS);
```
### 5. Looking for content in a directory using [Files.find](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#find(java.nio.file.Path,int,java.util.function.BiPredicate,java.nio.file.FileVisitOption...))

The Files.find method works the same as Files.walk except it takes a BiPredicate<Path,​BasicFileAttributes> as a parameter

```java
Path dir = Paths.get("D:/sources");
PathMatcher pathMatcher = FileSystems.getDefault().getPathMatcher("glob:**/*.java");

Stream<Path> paths = Files.find(dir, (path, attributes) -> pathMatcher.matches(path));
```

### 6. Warning about the weak consitency when exploring directories
Those streams are lazily built while exploring the directory tree, while working through
walking through the directory trees. It means that they are weakly consistent with the
file system. There is no lock put on the file system while exploring the directory tree.

The file system might change during the process. 

Even the directory we are currently exploring is deleted. In that case, we will have an exception.

### 7. Visiting Directory Trees
Visiting is different from Walking

Visiting a directory tree offers more control over the process:
* it can be interrupted
* it can skip elements based on filtering

The pattern uses the [Files.walkFileTree](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#walkFileTree(java.nio.file.Path,java.util.Set,int,java.nio.file.FileVisitor)) methods


### 3. How to filter the content of a tree with expressions and control over the depth of exploration?

### 4. How to visit a direcotry tree using the visitor pattern introduced in NIO2?