# [技术备忘录](../README.md) | [Java](README.md) | Lambda表达式，函数式接口

## 目录
  1. [java.util.function中的接口可分为哪四类？](#java-util-function-interface-categories)
  2. [Java8新增特性中三个有利于使用函数式接口的特性](#3-new-features-in-java-8-good-for-functional-interfaces)
  3. [java.util.function的各个函数式接口的方法名称是什么？](#method-names-of-java-util-function-interfaces)
  4. [java.util.function的函数式接口提供了哪些默认/静态方法？](#default-and-static-methods-provided-by-java-util-function-interfaces)
## 问题
### 1.java.util.function中的接口可分为哪四类？<a name="java-util-function-interface-categories"></a>[↑](#top)
java.util.function中一共有43个接口，可分为四类
    
    1. The Consumers(消费者接口)
         * A consumer consumes an object and does not return anything.

```java
public interface Consumer<T> {
    void accept(T t);
}
```

```java
Consumer<String printer = System.out::println;
```

```java
public interface BiConsumer<T, V> {
    void accept(T t, V v);
}
```

    2. The Supplier(生产者接口)
       * A supplier is the opposite of a consumer. It provides an object, takes not parameter.

```java
public interface Supplier<T> {
    T get();
}
```

```java
Supplier<Person> personSupplier = Person::new;
```

    3. The Functions(函数接口)
       * A function takes an object and returns another object

```java
public interface Function<T, R> {
    R apply(T t);
}
```

```java
Function<Person, Integer> ageMapper = Person::getAge;
```

```java
public interface BiFunction<T, V, R> {
    R apply(T t, V v);
}
```

```java
public interface UnaryOperator<T> extends Function<T, T> {}
```

```java
public interface BinaryOperator<T> extends BiFunction<T, T, T> {}
```


    4. The Predicates(判断条件接口)
       * A predicate takes an ojbect and returns a boolean

```java
public interface Predicate<T>  {
    boolean test(T t);
}
```

```java
Predicate<Person> ageGT20 = person -> person.getAge() > 20;
```

```java
public interface BiPredicate<T, U> {
    boolean test(T t, U u);
}
```

另外，针对基本类型(int, double, short等)，java.util.function中也定义了相应的接口。

类型 | 消费者接口(Consumer) | 生产者接口(Supplier) | 函数接口(Function) | 判断条件接口(Predicate)
------- | ------------------ | ------------------ | ----------------- | --------
引用类型 | [Consumer\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Consumer.html), [BiConsumer\<T,​U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiConsumer.html) | [Supplier\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Supplier.html) | [Function\<T,​R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Function.html), [BiFunction\<T,​U,​R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiFunction.html), [UnaryOperator\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/UnaryOperator.html), [BinaryOperator\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BinaryOperator.html) | [Predicate\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html), [BiPredicate\<T,​U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiPredicate.html)
int | [IntConsumer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntConsumer.html), [ObjIntConsumer\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ObjIntConsumer.html) | [IntSupplier](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntSupplier.html) | [IntFunction\<R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntFunction.html), [IntToLongFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntToLongFunction.html), [IntToDoubleFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntToDoubleFunction.html), [IntUnaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntUnaryOperator.html), [IntBinaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntBinaryOperator.html), [ToIntFunction\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToIntFunction.html), [ToIntBiFunction\<T,​U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToIntBiFunction.html) | [IntPredicate](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntPredicate.html)
long | [LongConsumer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongConsumer.html), [ObjLongConsumer\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ObjLongConsumer.html) | [LongSupplier](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongSupplier.html) | [LongFunction\<R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongFunction.html), [LongToIntFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongToIntFunction.html), [LongToDoubleFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongToDoubleFunction.html), [LongUnaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongUnaryOperator.html), [LongBinaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongBinaryOperator.html), [ToLongFunction\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToLongFunction.html), [ToLongBiFunction\<T,​U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToLongBiFunction.html) | [LongPredicate](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongPredicate.html)
double | [DoubleConsumer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleConsumer.html), [ObjDoubleConsumer\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ObjDoubleConsumer.html) | [DoubleSupplier](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleSupplier.html) | [DoubleFunction\<R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleFunction.html), [DoubleToIntFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleToIntFunction.html), [DoubleToLongFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleToLongFunction.html), [DoubleUnaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleUnaryOperator.html), [DoubleBinaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleBinaryOperator.html), [ToDoubleFunction\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToDoubleFunction.html), [ToDoubleBiFunction\<T,​U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToDoubleBiFunction.html) | [DoublePredicate](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoublePredicate.html)
boolean | N/A | [BooleanSupplier](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BooleanSupplier.html) | N/A | N/A

### 2. Java8新增特性中三个有利于使用函数式接口的特性<a name="3-new-features-in-java-8-good-for-functional-interfaces"></a>[↑](#top)
1. Lambda表达式: 用于实现函数式接口
2. 默认方法(default methods)：To chain the calls to certain functional interface.
3. 静态方法(static methods)：To build special instances of certain functional interface.

* **Default methods and static methods give the opportunity to build a new API for our old legacy code. Now we have the tools to add new functionalities to our old interfaces in our old code.**

### 3. java.util.function的各个函数式接口的方法名称是什么?<a name="method-names-of-java-util-function-interfaces"></a>

接口名称 | 方法名称
------- | -------
[Consumer\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Consumer.html), [BiConsumer\<T,​U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiConsumer.html), [IntConsumer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntConsumer.html), [LongConsumer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongConsumer.html), [DoubleConsumer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleConsumer.html), [ObjIntConsumer\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ObjIntConsumer.html), [ObjLongConsumer\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ObjLongConsumer.html), [ObjDoubleConsumer\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ObjDoubleConsumer.html) | accept
[Supplier\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Supplier.html) | get
[IntSupplier](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntSupplier.html) | getAsInt
[LongSupplier](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongSupplier.html) | 	getAsLong
[DoubleSupplier](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleSupplier.html) | getAsDouble
[BooleanSupplier](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BooleanSupplier.html) | getAsBoolean
[Function\<T,​R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Function.html), [UnaryOperator\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/UnaryOperator.html), [BiFunction\<T,​U,​R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiFunction.html), [BinaryOperator\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BinaryOperator.html), [IntFunction\<R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntFunction.html), [LongFunction\<R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongFunction.html), [DoubleFunction\<R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleFunction.html) | apply
[IntUnaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntUnaryOperator.html), [IntBinaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntBinaryOperator.html), [ToIntFunction\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToIntFunction.html), [ToIntBiFunction\<T,U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToIntBiFunction.html), [LongToIntFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongToIntFunction.html), [LongToIntFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongToIntFunction.html) | applyAsInt
[IntToLongFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntToLongFunction.html), [LongUnaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongUnaryOperator.html), [LongBinaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongBinaryOperator.html), [ToLongFunction\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToLongFunction.html), [ToLongBiFunction\<T,U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToLongBiFunction.html), [DoubleFunction\<R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleFunction.html) | applyAsLong
[IntToDoubleFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntToDoubleFunction.html), [LongToDoubleFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongToDoubleFunction.html), [DoubleUnaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleUnaryOperator.html), [DoubleBinaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleBinaryOperator.html), [ToDoubleFunction\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToDoubleFunction.html), [ToDoubleBiFunction\<T,U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToDoubleBiFunction.html) | applyAsDouble
[Predicate\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html), [BiPredicate\<T,​U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiPredicate.html), [IntPredicate](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntPredicate.html), [LongPredicate](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongPredicate.html), [DoublePredicate](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoublePredicate.html) | test

### 4. java.util.function的函数式接口提供了哪些默认/静态方法？<a name="default-and-static-methods-provided-by-java-util-function-interfaces"></a>[↑](#top)

1. 消费者接口

接口名称 | 默认方法 | 静态方法
------ | ------- | -------
[Consumer\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Consumer.html) | [andThen​(Consumer\<? super T\> after)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Consumer.html#andThen(java.util.function.Consumer)) | N/A
[BiConsumer\<T,​U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiConsumer.html) | [andThen​(BiConsumer\<? super T,​? super U\> after)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiConsumer.html#andThen(java.util.function.BiConsumer)) | N/A
[IntConsumer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntConsumer.html) | [andThen​(IntConsumer after)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntConsumer.html#andThen(java.util.function.IntConsumer)) | N/A
[LongConsumer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongConsumer.html) | [andThen​(LongConsumer after)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongConsumer.html#andThen(java.util.function.LongConsumer)) | N/A
[DoubleConsumer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleConsumer.html) | [andThen​(DoubleConsumer after)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleConsumer.html#andThen(java.util.function.DoubleConsumer)) | N/A
[ObjIntConsumer\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ObjIntConsumer.html) | N/A | N/A
[ObjLongConsumer\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ObjLongConsumer.html) | N/A | N/A
[ObjDoubleConsumer\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ObjDoubleConsumer.html) | N/A | N/A

2. 生产者接口

生产者类别的接口由于没有返回值，所以也没有任何默认方法、静态方法
接口名称 | 默认方法 | 静态方法
------ | ------- | -------
[Supplier\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Supplier.html) | N/A | N/A
[IntSupplier](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntSupplier.html) | N/A | N/A
[LongSupplier](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongSupplier.html) | N/A | N/A
[DoubleSupplier](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleSupplier.html) | N/A | N/A
[BooleanSupplier](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BooleanSupplier.html) | N/A | N/A

3. 引用类型函数接口

接口名称 | 默认方法 | 静态方法
------ | ------- | -------
[Function\<T,​R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Function.html) | [andThen​(Function\<? super R,​? extends V\> after)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Function.html#andThen(java.util.function.Function)), [compose​(Function\<? super V,​? extends T\> before)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Function.html#compose(java.util.function.Function)) | [identity()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Function.html#identity())
[UnaryOperator\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/UnaryOperator.html) | 继承自Function的andThen, compose | [identity()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/UnaryOperator.html#identity())
[BiFunction\<T,​U,​R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiFunction.html) | [andThen​(Function\<? super R,​? extends V\> after)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiFunction.html#andThen(java.util.function.Function)) | N/A
[BinaryOperator\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BinaryOperator.html) | 继承自BiFunction的andThen | [maxBy​(Comparator\<? super T\> comparator)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BinaryOperator.html#maxBy(java.util.Comparator)), [minBy​(Comparator\<? super T\> comparator)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BinaryOperator.html#minBy(java.util.Comparator))

4. 整型函数接口

接口名称 | 默认方法 | 静态方法
------ | ------- | -------
[IntFunction\<R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntFunction.html) | N/A | N/A
[IntToDoubleFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntToDoubleFunction.html) | N/A | N/A
[IntToLongFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntToLongFunction.html) | N/A | N/A
[IntUnaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntUnaryOperator.html) | [andThen​(IntUnaryOperator after)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntUnaryOperator.html#andThen(java.util.function.IntUnaryOperator)), [compose​(IntUnaryOperator before)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntUnaryOperator.html#compose(java.util.function.IntUnaryOperator)) | [identity()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntUnaryOperator.html#identity())
[IntBinaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntBinaryOperator.html) | N/A | N/A
[ToIntFunction\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToIntFunction.html) | N/A | N/A
[ToIntBiFunction\<T,U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToIntBiFunction.html) | N/A | N/A


5. 长整型函数接口

接口名称 | 默认方法 | 静态方法
------ | ------- | -------
[LongFunction\<R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongFunction.html) | N/A | N/A
[LongToDoubleFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongToDoubleFunction.html) | N/A | N/A
[LongToIntFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongToIntFunction.html) | N/A | N/A
[LongUnaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongUnaryOperator.html) | [andThen​(LongUnaryOperator after)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongUnaryOperator.html#andThen(java.util.function.LongUnaryOperator)), [compose​(LongUnaryOperator before)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongUnaryOperator.html#compose(java.util.function.LongUnaryOperator)) | [identity()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongUnaryOperator.html#identity())
[LongBinaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongBinaryOperator.html) | N/A | N/A
[ToLongFunction\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToLongFunction.html) | N/A | N/A
[ToLongBiFunction\<T,U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToLongBiFunction.html) | N/A | N/A

6. 双精度型函数接口

接口名称 | 默认方法 | 静态方法
------ | ------- | -------
[DoubleFunction\<R\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleFunction.html) | N/A | N/A
[DoubleToLongFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleToLongFunction.html) | N/A | N/A
[DoubleToIntFunction](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleToIntFunction.html) | N/A | N/A
[DoubleUnaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleUnaryOperator.html) | [andThen​(DoubleUnaryOperator after)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleUnaryOperator.html#andThen(java.util.function.DoubleUnaryOperator)), [compose​(DoubleUnaryOperator before)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleUnaryOperator.html#compose(java.util.function.DoubleUnaryOperator)) | [identity()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleUnaryOperator.html#identity())
[DoubleBinaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoubleBinaryOperator.html) | N/A | N/A
[ToDoubleFunction\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToDoubleFunction.html) | N/A | N/A
[ToDoubleBiFunction\<T,U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/ToDoubleBiFunction.html) | N/A | N/A

7. 判断条件接口

接口名称 | 默认方法 | 静态方法
------ | ------- | -------
[Predicate\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html) | [and​(Predicate\<? super T\> other)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html#and(java.util.function.Predicate)), [or​(Predicate\<? super T\> other)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html#or(java.util.function.Predicate)), [negate()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html#negate()) | [isEqual​(Object targetRef)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html#isEqual(java.lang.Object)), [not​(Predicate\<? super T\> target)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html#not(java.util.function.Predicate))
[BiPredicate\<T,​U\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiPredicate.html) | [and​(BiPredicate\<? super T,​? super U\> other)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiPredicate.html#and(java.util.function.BiPredicate)), [or​(BiPredicate\<? super T,​? super U\> other)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiPredicate.html#or(java.util.function.BiPredicate)), [negate()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BiPredicate.html#negate()) | N/A
[IntPredicate](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntPredicate.html) | [and​(IntPredicate other)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntPredicate.html#and(java.util.function.IntPredicate)), [or​(IntPredicate other)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntPredicate.html#or(java.util.function.IntPredicate)), [negate()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/IntPredicate.html#negate()) | N/A
[LongPredicate](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongPredicate.html) | [and​(LongPredicate other)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongPredicate.html#and(java.util.function.LongPredicate)), [or​(LongPredicate other)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongPredicate.html#or(java.util.function.LongPredicate)), [negate()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/LongPredicate.html#negate()) | N/A
[DoublePredicate](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoublePredicate.html) | [and​(DoublePredicate other)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoublePredicate.html#and(java.util.function.DoublePredicate)), [or​(DoublePredicate other)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoublePredicate.html#or(java.util.function.DoublePredicate)), [negate()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/DoublePredicate.html#negate()) | N/A

