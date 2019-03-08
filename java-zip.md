# Java Zip/GZip 备忘录
## 目录
  1. [如何向一个GZip文件写入数据？](#write-to-gzip)
  2. [如何从一个GZip文件读取数据？](#read-from-gzip)
## 问题
### 1.如何向一个GZip文件写入数据？<a name="write-to-gzip"></a>[↑](#top)
下面的代码在files目录中创建gzip格式的压缩文件file.bin.gz，并把从1到100的
32位整数写入其中。其中利用Decorator模式把一个OutputStream封装在GZIPOutputStream中，然后再用DataOutputStream封装GZIPOutputStream。业务代码向DataOutputStream写入数据，并不会感知到GZIPOutputStream的存在。对业务代码来说，写入普通文件和写入GZip文件没有区别。
```java
try(DataOutputStream dos = new DataOutputStream(new GZIPOutputStream(Files.newOutputStream(
    Paths.get("files/file.bin.gz"),
    StandardOpenOption.CREATE
)))){
    IntStream.range(0, 100).forEach(i -> {
        try{ 
            dos.writeInt(i);
        } catch(IOException ex){
            ex.printStackTrace();
        }
    });
}
```
### 2.如何从一个GZip文件读取数据？<a name="read-from-gzip"></a>[↑](#top)
下面的代码从gzip压缩文件files/file.bin.gz中读取上面的代码写入的100个整数并将其以逗号做分隔符拼接为一个字符串后输出到标准输出。
```java
try(DataInputStream dis = new DataInputStream(new GZIPInputStream(Files.newInputStream(
    Paths.get("files/file.bin.gz"),
    StandardOpenOption.READ
)))){
    java.util.List<Integer> ints = new ArrayList<>();
    while(true){
        int i;
        try{
            i = dis.readInt();
            ints.add(i);
        } catch(EOFException ignore){
            break;
        }
        ints.add(i);
    }
    String result = ints.stream()
        .map(Object::toString)
        .collect(Collectors.joining(", "));
    System.out.println(result);
}
```