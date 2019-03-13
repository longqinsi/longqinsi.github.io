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
- | 名称    | 发布年份 | JDK版本
- | ------ | ------ | -------
1 | Java I/O | 1996 | Java 1
2 | Java NIO | 2002 | Java 4
3 | Java NIO 2 | 2011 | Java 7  