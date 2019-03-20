# [技术备忘录](../README.md) | [Java](README.md) | Java8流式数据处理
## 目录
  1. [如何把一个流的数据拼接为字符串？](#join-stream)
  2. [map(映射)/filter(过滤)/reduce(规约)](#map-filter-reduce)
  3. [Java 8 为什么要引入 Stream API?](#why-java-8-introduces-stream)
  4. [Stream 的关键特征？](#characteristics-of-stream)
  5. [Intermediate & Termianl Calls](#intermediate-and-termianl-calls)
  6. [How to recognize a terminal call?](#how-to-recognize-terminal-call)
  7. [Select ranges of data in streams](#select-range-in-stream)
  8. [Match Reductions](#match-reductions)
  8. [Find Reductions](#find-reductions)
  8. [Reduce Reductions](#reduce-reductions)
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

### 2. map(映射)/filter(过滤)/reduce(规约)<a name="map-filter-reduce"></a>[↑](#top)
* 在 Java 8 中，可以使用 Lambda 表达式和集合实现 map/filter/reduce 算法。
* 在 map/filter/reduce 这三个步骤中，reduce的坑最多，需要投入更多的精力去学习。
* 例子：计算一群人中所有年龄大于20岁的人的平均年龄
  * 在Java 8以前，实现的代码可能如下：
    ```java
    List<Person> people = ...;
    int sum = 0;
    int count = 0;
    for (Person p : people) {
      if ( p.getAge() > 20) {
        sum += p.getAge();
        count++;
      }
    }
    int average = 0;
    if (count > 0)
      average = sum / count;
    ```
  * 对上述例子使用 map/filter/reduce 的分析如下图示意：
    ![map](/images/avg-age-of-people-older-than-20-map.png)
    ![filter](/images/avg-age-of-people-older-than-20-filter.png)
    ![reduce](/images/avg-age-of-people-older-than-20-reduce.png)

  * 使用 Java 7 的模式——创建帮助类 Lists，来实现 map/filter/reduce
    ```java
    List<Person> people = ...;
    List<Integer> ages     = Lists.map(people, Person::getAge);
    List<Integer> agesGT20 = Lists.filter(ages, age -> age > 20);
    int sum                = Lists.reduce(agesGT20, Integer::sum);
    ```

  * 对 reduce 这一步可以在多核CPU上并行运算，其条件是 reduce 对应的运算必须满足结合律（associativity）。
    * 下面是一些满足或不满足结合律的二元操作符的例子
      * BinaryOperator<Integer> op1 = (i1, i2) -> i1 + i2;
        * 整数的加法满足结合律 —— 对一系列整数的求和运算可以并行化
      * BinaryOperator<Integer> op2 = (i1, i2) -> Integer.max(i1, i2)
        * 求两个整数中的最大值的运算满足结合律 —— 计算一系列整数的最大值的运行可以并行化
      * BinaryOperator<Integer> op3 = (i1, i2) -> i1 * i1 + i2 * i2;
        * 求两个整数的平方和的运算不满足结合律
      * BinaryOperator<Integer> op4 = (i1, i2) -> i1;
        * 取两个整数中的第一个的运算满足结合律 —— 获取一系列整数中的第一个整数的运算可以并行化
      * BinaryOperator<Integer> op5 = (i1, i2) -> (i1 + i2)/2;
        * 求两个整数的平均值的运算不满足结合律 —— 求一系列整数的平均值的运算无法并行化
  * 如果把不满足结合律的运算并行化，对于同一个集合，可能每次计算得到的结果都会不同。
  * 对于 reduce(规约) 运算，还要求该运算有一个幂等元(identity element)。对于 取左运算(i1, i2) -> i1 运算，就没有这样的幂等元
    * 对于这种情况，可以用 Java 8 提供的 [Optional\<T\>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html) 对类型做封装，然后用 Optional.empty() 作幂等元。例子如下：
      ```java
      public static <T> T reduce(
            List<T> values,
            T valueOfEmpty,
            BinaryOperator<T> reduction) {

        T result = valueOfEmpty;
        for (T value : values) {
            result = reduction.apply(result, value);
        }
        return result;
      }

      public static void main(String[] args) {

          List<Optional<Integer>> ints = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
                                              .map(Optional::of).collect(Collectors.toList());

          List<Optional<Integer>> ints1 = Stream.of(1, 2, 3, 4)
                  .map(Optional::of).collect(Collectors.toList());
          List<Optional<Integer>> ints2 = Stream.of(5, 6, 7, 8, 9)
                  .map(Optional::of).collect(Collectors.toList());

          BinaryOperator<Optional<Integer>> op = (i1, i2) -> i1.isEmpty() ? i2 : i1;

          Optional<Integer> reduction1 = reduce(ints1, Optional.empty(), op);
          Optional<Integer> reduction2 = reduce(ints2, Optional.empty(), op);
          Optional<Integer> reductionParallel = reduce(Arrays.asList(reduction1, reduction2),   Optional.empty(), op);
          Optional<Integer> reduction = reduce(ints, Optional.empty(), op);

          System.out.println("Parallel reduction : " + reductionParallel);
          System.out.println("Reduction : " + reduction);
      }
      ```
      其运行结果如下：
      ```bash
      Parallel reduction : Optional[1]
      Reduction : Optional[1]
      ```

### 3. Java 8 为什么要引入 Stream API?<a name="why-java-8-introduces-stream"></a>[↑](#top)
* 这是为了避免在执行 map/filter/reduce 的过程中构建中间集合。
  
  * 如果在集合上直接添加 map/filter/reduce 方法，让这些方法返回新的集合，就会造成产生很多只被使用一次的中间集合，这会产生不必要的内存浪费，同时对这些临时对象的垃圾回收又会对 CPU 造成不必要的压力。

### 4. Stream 的关键特征？<a name="characteristics-of-stream"></a>[↑](#top)
* A Stream **does not hold** any data. 
  > A collection holds its data.
  * It pulls the data it processes from a source
    * The source could be a collection, it might also be something else.
    * The stream will connect to that source, consume the data from that source,
      and processes the elements in it.
  * There is no data in a Stream, but there is data in a collection.

* A Stream **does not modify** the data it processes.  
  > A collection can modify the data it holds.
  * Because we want to process the data in parallel with no visibility issues
    or synchronization issues that could lead to bad performance.
  * This is enforced by Java compiler or JDK, but a contract.

* The source may be **unbounded**.
  > A collection holds a known and finite amount of elements.
  * It means that the stream can process as many data as we need.
  * A source may be unbound because it is infinite.
  * But most of the time, it only means that the size of this source is unknown
    at build time.
    * Suppose the source is the lines of a text file with a very big size. I know
    the exact size of this text file, but if I do not open it and go through all
    of its content, I will not be able to know the number of lines inside this
    text file.

### 5. Intermediate & Termianl Calls<a name="intermediate-and-termianl-calls"></a>[↑](#top)
* For instance, [Stream.forEach()](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#forEach(java.util.function.Consumer)) is a terminal operation, but [Stream.peek()](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#peek(java.util.function.Consumer)) is an termediate operation.
  * Terminal operation
    * A termianl operation triggers the processing of a Stream.
      * No terminal operation = no data is ever processed.
  * Intermediate operation does not trigger anything.

### 6. How to recognize a terminal call?<a name="how-to-recognize-terminal-call"></a>[↑](#top)
1) Read the Javadoc!
2) A trick: 
   1) A call that returns a String is an intermediate call
   2) A call that returns something else, or void is a termianl call.

### 7. Select ranges of data in streams<a name="select-range-in-stream"></a>[↑](#top)
Using [skip()](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#skip(long)) and [limit()]https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#limit(long))

### 8. Match Reductions<a name="match-reductions"></a>[↑](#top)
There are three types of matchers. They are terminal operations that return a boolean.
These three matchers may not evaluate the predicate for all the elements, so they are called short-circuiting terminal operations.
* [anyMatch()](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#anyMatch(java.util.function.Predicate))
  * ```boolean anyMatch​(Predicate<? super T> predicate)```
* [allMatch()](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#allMatch(java.util.function.Predicate))
  * ```boolean allMatch​(Predicate<? super T> predicate)```
* [noneMatch()](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#noneMatch(java.util.function.Predicate))
  * ```boolean noneMatch​(Predicate<? super T> predicate)```

### 9. Find Reductions<a name="find-reductions"></a>[↑](#top)
There are two types of find reduction:

* [findFirst()](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#findFirst())
  * ```Optional<T> findFirst()```
  > Returns an Optional describing the first element of this stream, or an empty Optional if the stream is empty. If the stream has no encounter order, then any element may be returned.
  > 
  > This is a short-circuiting terminal operation.
* [findAny()](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#findAny())
  * ```Optional<T> findAny()```
  > Returns an Optional describing some element of the stream, or an empty Optional if the stream is empty.
  >
  > This is a short-circuiting terminal operation.
  >
  > The behavior of this operation is explicitly nondeterministic; it is free to select any element in the stream. This is to allow for maximal performance in parallel operations; the cost is that multiple invocations on the same source may not return the same result. (If a stable result is desired, use findFirst() instead.)

### 10. Reduce Reductions<a name="reduce-reductions"></a>[↑](#top)
There are three types of reduce reduction
* If no identity element is provided, then an Optional is returned.
  * [```Optional<T> reduce​(BinaryOperator<T> accumulator```)](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#reduce(java.util.function.BinaryOperator))
  
  ```java
  List<Person> people = ...;
  Optional<Integer> sumOfAges = 
                  people.stream().mapToInt(Person::getAge)
                        .reduce((Integer::sum);
  ```

* If an identity element is provided, then a result of the element type of the stream is returned.
  * [```reduce​(T identity, BinaryOperator<T> accumulator)```](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#reduce(T,java.util.function.BinaryOperator))

  ```java
  List<Person> people = ...;
  Optional<Integer> sumOfAges = 
                  people.stream()
                        .reduce(0, (p1, p2) -> p1.getAge() + p2.getAge());
  ```

* Third version of reduce(): used in parallel operations. It needs you provide 
  the identity element, accumalator, and combiner.
  * [```<U> U reduce​(U identity, BiFunction<U,​? super T,​U> accumulator, BinaryOperator<U> combiner)```](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#reduce(U,java.util.function.BiFunction,java.util.function.BinaryOperator))

  ```java
  List<Person> people = ...;

  List<Integer> ages =
  people.stream()
        .reduce(
            new ArrayList<integer>(),
            (list, p) -> { list.add(p.getAge()); return list; },
            (list1, list2) -> { list1.addAll(list2); return list1; }
        );
  ```
