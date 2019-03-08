# Java IO 备忘录

## 目录
  1. [如何递归获取指定目录下的所有子目录和文件？](#recursively-ls)
  2. [如何用工厂模式创建BufferedWritter？](#create-buffered-writer)
  3. [如何打开文件并附加到文件末尾？](#append-to-file)
  4. [如何在关闭文件时删除它？](#delete-on-close)
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
