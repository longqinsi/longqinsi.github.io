# [Java 备忘录](README.md) | 序列化/反序列化
## 目录
  1. [如何序列化对象？](#serialize-object)
  2. [如何通过反序列化创建对象？](#create-object-by-deserialization)
  3. [如何使Intellij IDEA为标记了Serializable的类自动生成serialVersionUID字段？](#idea-serial-version-uid)
  4. [如何使某个字段不被序列化？](#exclude-field)  

## 问题
### 1.如何序列化对象？<a name="serialize-object"></a>[↑](#top)
下面的代码把Person类的对象person1和person2序列化后写入文件files/people.bin中。
```java
Person person1 = new Person("Charles", 20);
Person person2 = new Person("James", 25);
try(
  ObjectOutputStream oos = new ObjectOutputStream(
  Files.newOutputStream(
    Paths.get("files/people.bin"),
    StandardOpenOption.CREATE
  ))){
  oos.writeObject(person1);
  oos.writeObject(person2);
  // 在文件末尾写入一个null，作为结束标识符，当读取文件时，
  // 如果读到null，则表示已到文件末尾，停止读取。
  oos.writeObject(null);
}
```
备注：Person类的定义如下：
```java
public class Person implements Serializable {
	private static final long serialVersionUID = -2654901401681242080L;
	private final String name;
	private final int age;
	public Person(String name, int age) {
		this.name = name;
		this.age = age;
	}
	public String getName() { return name; }
	public int getAge() { return age; }
}
```
### 2.如何通过反序列化创建对象？<a name="create-object-by-deserialization"></a>[↑](#top)
下面的代码从问题1所创建的文件中反序列化得到对象person2，并将其姓名和年龄输出到命令行。
```java
try(
  InputStream fis = Files.newInputStream(Paths.get("files/people.bin"));
  ObjectInputStream ois = new ObjectInputStream(fis)){

  while(true){
    Person person = (Person)ois.readObject();
    // 如果读取的对象为null，表示到了文件末尾，停止读取。也可以通过捕获
    // EOFException来探测是否到文件末尾。但捕获异常对性能有影响。
    if(person == null){
      break;
    }
    System.out.printf("name: %s, age: %d%n", person.getName(),    person.getAge());
  }
}
```
其输出如下：
```
name: Charles, age: 20
name: James, age: 25
```
### 3.如何使Intellij IDEA为标记了Serializable的类自动生成serialVersionUID字段？<a name="idea-serial-version-uid"></a>[↑](#top)
Go to menu File → Settings → Editor → Inspections → Java → Serialization issues → Serializable class without 'serialVersionUID'` enabled, the class you provide will give me warnings.

![Intellij IDEA对实现了Serializable接口却没有定义serialVersionUID字段的类的警告](/images/idea-serialization-warning.png)
### 4.如何使某个字段不被序列化？<a name="exclude-field"></a>[↑](#top)
用transient关键字标记这个字段。
下面代码中，Person类的address字段就不会被序列化。
```java
public class Person implements Serializable {

	private String name;
	private int age;
	
	private transient Address address;
}
```