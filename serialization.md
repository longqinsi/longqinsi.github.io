# [Java 备忘录](README.md) | 序列化/反序列化
## 目录
  1. [如何使Intellij IDEA为标记了Serializable的类自动生成serialVersionUID字段？](#idea-serial-version-uid)
  2. [如何使某个字段不被序列化？](#exclude-field)  

## 问题
### 1.如何使Intellij IDEA为标记了Serializable的类自动生成serialVersionUID字段？<a name="idea-serial-version-uid"></a>[↑](#top)
Go to menu File → Settings... → Inspections → Serialization issues → Serializable class without 'serialVersionUID'` enabled, the class you provide give me warnings.

![Intellij IDEA对实现了Serializable接口却没有定义serialVersionUID字段的类的警告](/images/idea-serialization-1.png)
### 2.如何使某个字段不被序列化？<a name="exclude-field"></a>[↑](#top)
用transient关键字标记这个字段。
下面代码中，Person类的address字段就不会被序列化。
```java
public class Person implements Serializable {

	private String name;
	private int age;
	
	private transient Address address;
}
```