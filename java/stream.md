# [Java](README.md) | Java8流式数据处理
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

### 2. map(映射)/filter(过滤)/reduce(规约)
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

### 3. Java 8 为什么要引入 Stream API?
这是为了有效地实现 map/filter/reduce 算法。如果在集合上直接添加 map/filter/reduce 方法，
让这些方法返回新的集合，就会造成产生很多只被使用一次的中间集合，这会产生不必要的内存浪费，同时对这些
临时对象的垃圾回收又会对 CPU 造成不必要的压力。