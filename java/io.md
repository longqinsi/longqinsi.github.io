# [技术备忘录](../README.md) | [Java](README.md) | Input/Output

## 目录
  1. [如何递归获取指定目录下的所有子目录和文件？](#recursively-ls)
  2. [如何用工厂模式创建BufferedWritter？](#create-buffered-writer)
  3. [如何打开文件并附加到文件末尾？](#append-to-file)
  4. [如何在关闭文件时删除它？](#delete-on-close)
  5. [如何在printf中只传入参数一次，却使用多次？](#printf-argument-index)
  6. [What's a hybrid stream?](#hybrid-stream)

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
There is [a code example](https://github.com/longqinsi/HybridStreams), which is taken from the last module [Dealing with Hybrid Streams of Text and Bytes](https://app.pluralsight.com/player?course=java-fundamentals-input-output&author=jose-paumard&name=dfbf5974-c66a-479b-89ed-87d01d4010d7&clip=0&mode=live) of [Jose Paumard](http://blog.paumard.org/en/) 's [pluralsight](https://www.pluralsight.com/) course [Java Fundamentals: Input/Output](https://app.pluralsight.com/library/courses/java-fundamentals-input-output).
