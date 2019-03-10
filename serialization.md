# [Java 备忘录](README.md) | 序列化/反序列化
## 目录
  1. [如何序列化对象？](#serialize-object)
  2. [如何通过反序列化创建对象？](#create-object-by-deserialization)
  3. [如何使Intellij IDEA为标记了Serializable的类自动生成serialVersionUID字段？](#idea-serial-version-uid)
  4. [如何使某个字段不被序列化？](#exclude-field) 
  5. [有哪几种方法可以覆盖JVM的默认序列化机制？](#override-default-mechanism)  

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
### 5.有哪几种方法可以覆盖JVM的默认序列化机制？<a name="override-default-mechanism"></a>[↑](#top)
方法1：提供一对writeObject()/readObject()方法
下面的Person类在writeObject方法中用DataOutputStream类封装ObjectOutputStream，把name和age用"::"分隔后拼接为一个字符串序列化，并给name和age字段都加上了transient关键字，这样可以有效减少序列化后的文件大小。
在readObject中对应地用DataInputStream封装ObjectInputStream进行读取。
```java
public class Person implements Serializable {
	private static final long serialVersionUID = -2654901401681242080L;
	private transient String name;
	private transient int age;
	public Person(String name, int age) {
		this.name = name;
		this.age = age;
	}

	private void writeObject(ObjectOutputStream oos) throws Exception {
		DataOutputStream dos = new DataOutputStream(oos);
		dos.writeUTF(name + "::" + age);
	}

	private void readObject(ObjectInputStream ois) throws Exception {
		DataInputStream dis = new DataInputStream(ois);
		String[] contents = dis.readUTF().split("::");
		this.name = contents[0];
		this.age = Integer.parseInt(contents[1]);
	}

	public String getName() { return name; }
	public int getAge() { return age; }
}
```
**备注：如果类Employee是Person的子类，在序列化/反序列化Employee的对象时，Employee和Person的readObject/writeObject方法都会被调用，Employee类的readObject/writeObject方法只需要负责它本身定义的字段（例如salary工资）的序列化/反序列化。**

方法2：实现Externalizable接口
Externalizable接口的定义如下：
```java
public interface Externalizable {

  void writeExternal(ObjectOutput out) throws IOException;

  void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}
```
和第一种方法的不同是：

1）writeExternal/readExternal不仅要序列化/反序列化自身的字段，还需要序列化/反序列化基类字段（如果有基类）。

2）writeExternal/readExternal方法是公开(public)的，而writeObject/readObject方法是私有(private)的。

使用场景：发送方只序列化对象的主键，而不序列化其状态；接收方通过对象的类和主键重建其状态（状态可能保存在数据库中）。


方法3：使用代理类进行序列化/反序列化

1)在需要序列化的类A提供一个 **私有(private)** 的writeReplace方法，返回一个代理类B的对象。

2)在代理类B中通过一个 **私有(private)** 的readResolve方法返回类A的对象。

在序列化流中存储的是代理类B的对象，但这对于调用方是透明的。

使用场景：
It has serveral uses. One of them is to handle legacy applications.
用于序列化可序列化类的不同版本。Supposed that on a disk you have a collections of file with all versions of all the classes. You can just add this readResolve method in these old classes to provide new objects to your application. 

需要序列化的类：
```java
public class Person implements Serializable {

	private static final long serialVersionUID = -2654901401681242080L;

	private String name;

	private int age;

	public Person(String name, int age) {
		this.name = name;
		this.age = age;
	}

	private Object writeReplace() throws ObjectStreamException{
		return new PersonProxy(name + "::" + age);
	}

	public String getName() { return name; }
	public int getAge() { return age; }
}
```
代理类：
```java
public class PersonProxy implements Serializable {

	private static final long serialVersionUID = 1695295410281921902L;

	private String name;

	public PersonProxy() {
	}
	
	private Object readResolve() throws ObjectStreamException {
		
		String[] strings = this.name.split("::");
		String name = strings[0];
		int age = Integer.parseInt(strings[1]);
		
		return new Person(name, age);
	}

	public PersonProxy(String name) {
		this.name = name;
	}
	
	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	@Override
	public String toString() {
		return "PersonProxy [name=" + name + "]";
	}
}
```