# Java Zip/GZip 备忘录
## 目录
  1. [如何向一个GZip文件写入数据？](#write-to-gzip)
  2. [如何从一个GZip文件读取数据？](#read-from-gzip)
  3. [如何提高向GZIP/ZIP文件写入数据的效率？](#performant-zip-write)
  4. [如何向zip文件写入数据？](#write-to-zip)
  5. [如何从zip文件读取数据？](#read-from-zip)

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
### 3.如何提高向GZIP/ZIP文件写入数据的效率？<a name="performant-zip-write"></a>[↑](#top)
Creating GZIP/ZIP streams in memory, and then flushing them
to the disk or network once the compression is done can lead
to much better performance than to use simple buffered OutputStreams.
### 4.如何向zip文件写入数据？<a name="write-to-zip"></a>[↑](#top)
下面的代码在zip压缩包files/archive.zip中创建了files目录，然后在files目录下创建了两个文件data1.bin和data2.bin，并向两个文件中分别写入了两个32位整数。
使用ZipEntry向zip输出流中添加目录/文件。
在添加文件ZipEntry后，再利用外层的DataOutputStream写入数据，在向文件写入数据完成后，调用ZipEntry.closeEntry方法关闭这个ZipEntry。
```java
try(ZipOutputStream zos = new ZipOutputStream(
        Files.newOutputStream(
            Paths.get("files/archive.zip"),
            StandardOpenOption.CREATE
        ), StandardCharsets.UTF_8);
    DataOutputStream dos = new DataOutputStream(zos);
){
    ZipEntry dirEntry = new ZipEntry("files/");
    zos.putNextEntry(dirEntry);

    ZipEntry fileEntry1 = new ZipEntry("files/data1.bin");
    zos.putNextEntry(fileEntry1);

    dos.writeInt(1);
    dos.writeInt(2);

    zos.closeEntry();

    ZipEntry fileEntry2 = new ZipEntry("files/data2.bin");
    zos.putNextEntry(fileEntry2);

    dos.writeInt(3);
    dos.writeInt(4);

    zos.closeEntry();
}
```
### 5.如何从zip文件读取数据？<a name="read-from-zip"></a>[↑](#top)
下面的代码从问题4中创建的zip文件中读取目录和文件，并打印其名称。对于文件，还要读读取其中包含的32位整数。
```java
try(ZipInputStream zis = new ZipInputStream(
        Files.newInputStream(
            Paths.get("files/archive.zip"),
            StandardOpenOption.READ
        ), StandardCharsets.UTF_8);
    DataInputStream dis = new DataInputStream(zis);
){
    for(ZipEntry nextEntry = zis.getNextEntry(); nextEntry != null; nextEntry = zis.getNextEntry()) {
        if (nextEntry.isDirectory()){
            // deal with directories
            System.out.printf("目录：%s%n", nextEntry.getName());
        } else {
            System.out.printf("文件：%s%n", nextEntry.getName());
            while(true){
                try{
                    int i = dis.readInt();
                    System.out.println(i);
                } catch (EOFException e){
                    break;
                }
            }
        }
    }
}
```