# [Java语法、Java基础类库和Java虚拟机](README.md) | Java8流式数据处理
## 目录
  1. [如何把一个流的数据拼接为字符串？](#join-stream)

## 问题
### 1.如何把一个流的数据拼接为字符串？<a name="join-stream"></a>[↑](#top)
下面的代码把从0到9的整数用逗号分隔，拼成一个字符串。
```java
IntStream.range(0, 10)
   .mapToObj(String::valueOf)
   .collect(Collectors.joining(", "));
```
结果是
```
"0, 1, 2, 3, 4, 5, 6, 7, 8, 9"
```
