# Java IO 备忘录

## 目录
  1. [如何递归获取指定目录下的所有子目录和文件？](#recursively-ls)
  1. [如何用工厂模式创建BufferedWritter？](#create-buffered-writer)
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
### 1.如何用工厂模式创建BufferedWritter？<a name="create-buffered-writer"></a>[↑](#top)

调用Files.newBufferedWriter方法
```java
Path path = Paths.get("files/data.txt");
Files.newBufferedWriter(path);
```