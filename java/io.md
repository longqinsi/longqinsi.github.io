# [技术备忘录](../README.md) | [Java](README.md) | Input/Output

## 目录
  1. [如何递归获取指定目录下的所有子目录和文件？](#recursively-ls)
  2. [如何用工厂模式创建BufferedWritter？](#create-buffered-writer)
  3. [如何打开文件并附加到文件末尾？](#append-to-file)
  4. [如何在关闭文件时删除它？](#delete-on-close)
  5. [如何在printf中只传入参数一次，却使用多次？](#printf-argument-index)
  6. [What's a hybrid stream?](#hybrid-stream)
  7. [What's marking, resetting and skipping of the Reader api, and how to use them?](#reader-mark-reset-skip)
## 问题
### 1.如何递归获取指定目录下的所有子目录和文件？<a name="recursively-ls"></a>[↑](#top)

用Files.walk方法获取子目录和文件的遍历流然后用forEach处理单个子目录/文件
```java
Files.walk(Paths.get(".")).forEach(System.out::println);
```
代码的输出如下：
```bash
.
./files
./src
./src/org
./src/org/paumard
./src/org/paumard/io
./src/org/paumard/io/WritingCharacters.java
```
### 2.如何用工厂模式创建BufferedWritter？<a name="create-buffered-writer"></a>[↑](#top)
调用Files.newBufferedWriter方法
```java
Path path = Paths.get("files/data.txt");
Files.newBufferedWriter(path);
```
### 3.如何打开文件并附加到文件末尾？<a name="append-to-file"></a>[↑](#top)
在打开文件时设置 StandardOpenOption.APPEND 选项
```java
Path path = Paths.get("files/data.txt");
Files.newBufferedWriter(path, StandardOpenOption.APPEND);
```
**备注：如果不设置 APPEND 选项，默认是覆盖原有内容**

### 4.如何在关闭文件时删除它？<a name="delete-on-close"></a>[↑](#top)
在打开文件时设置 StandardOpenOption.DELETE_ON_CLOSE 选项
```java
Path path = Paths.get("files/data.txt");
Files.newBufferedWriter(path, StandardOpenOption.DELETE_ON_CLOSE );
```

### 5.如何在printf中只传入参数一次，却使用多次？<a name="printf-argument-index"></a>[↑](#top)
在格式占位符%后加参数序号（从1开始）和一个美元符号$
```java
Calendar calendar = GregorianCalendar.getInstance();
calendar.set(1969, 6, 20);
System.out.printf("人类于%1$tY年%1$tm月%1$te日登上月球。", calendar);
```
程序输出：
```bash
Man walked on the moon on: 07 20 1969
```

### 6. What's a hybrid stream?<a name="hybrid-stream"></a>[↑](#top)
A hybrid stream is just a regular character stream, and a binary stream that are open together, on the same binary stream.
There is a code example [HybridStreams](https://github.com/longqinsi/HybridStreams), taken from José Paumard.
Please refer to [WritingHybridStream.java](https://github.com/longqinsi/HybridStreams/blob/master/src/org/paumard/io/WritingHybridStream.java) for how to write to hybrid streams, and [ReadingHybridStream.java](https://github.com/longqinsi/HybridStreams/blob/master/src/org/paumard/io/ReadingHybridStream.java) for how to read from hybrid streams.

### 7. What's marking, resetting and skipping of the Reader api, and how to use them?<a name="reader-mark-reset-skip"></a>[↑](#top)
Readers support **[marking](https://docs.oracle.com/javase/8/docs/api/java/io/Reader.html#mark-int-)**, **[resetting](https://docs.oracle.com/javase/8/docs/api/java/io/Reader.html#reset--)**, and **[skipping](https://docs.oracle.com/javase/8/docs/api/java/io/Reader.html#skip-long-)**. 
A reader **can** skip elements. It is supported by all the implementations of the Reader abstract class. 
A reader **may** support reset, which is go back to the beginning. It is not necessarily supported by all the implementations of the Reader abstract class, but we cannot test it is supported or not.
A Reader **may** support the marking of the character stream, and we can test if the mark operation is supported or not with the [markSupported()](https://docs.oracle.com/javase/8/docs/api/java/io/Reader.html#markSupported--) method call. 
How to use mark, reset and skip?
A [mark()](https://docs.oracle.com/javase/8/docs/api/java/io/Reader.html#mark-int-) call just puts a flag on a given element of the character stream. If I call [reset()](https://docs.oracle.com/javase/8/docs/api/java/io/Reader.html#reset--), it will rewind to the previously marked element if there is one. If it's not the case, it will rewind to the beginning of the stream. And a [skip()](https://docs.oracle.com/javase/8/docs/api/java/io/Reader.html#skip-long-) call just skips the next element of the character stream.
Refer to [ReadingHybridStream.java](https://github.com/longqinsi/HybridStreams/blob/master/src/org/paumard/io/ReadingHybridStream.java) for an exmaple.