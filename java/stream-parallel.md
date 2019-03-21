# [技术备忘录](../README.md) | [Java](README.md) | Java8流式数据处理-并行处理
## 目录
  1. [Parallel streams](#parallel-streams)
  2. [Stateful & stateless operations](#stateful-and-stateless-operations)
  3. [Parallel reductions](#parallel-reductions)
## 问题
### 1. Parallel streams<a name="parallel-streams"></a>[↑](#top)
* The difference between going parallel and going multithread.

* Why going parallel?
  * To allow for faster computation
  * To leverage the multicore CPU

* Multithread ≠ parallel
  * Multithread means that each data processing is conducted in its own thread.
  * Parallel processing means that a given processing, a given data pipeline, is computed among
    more than one thread.
  > Take quick sorting for instance:
  > 
  > If I am in a multithread environment, each thread will be able to sort one given set of integers. 
  > 
  > If I am in a parallel world, it means that a pool of thread will be able to sort one set of integers.

  > In the case of quicksort, the algorithm that will run in one thread, is not the same as the
  > one that will run over more than one thread.
* Multithread: one process = one thread, so many processes at the same time
  * Web server, and an application server are examples.
  * Problems: 
    * race condition
      * Many threads are trying to read and write the same variable or the same object.
    * thread synchronization
    * variable visibility
* Parallel: one process = many threads, to go faster
  * Problems: algorithm, data distribution among the CPU cores, the balancing of the CPU loads.

* Tools for parallel processing in the JDK
  * In Java 6 and before: none!
    * Everything has to be done by hand.
  * In Java 7
    * The fork/join framework
      > It is a very technical framework, quite hard to master, even harder to debug when
      > I have problems, issues in my computations. So it is merely a low level tool that
      > I cannot really use efficiently in my applications.
    * A third party API: parallel arrays (Java 6 compatible, with its own embedded fork/join)
  * In Java 8: parallel streams
    * Much easier and safer to use
      > There are situations where it is not safe.
      > 
      > It is built on the fork/join framework. So we can say that as a nice way to access
      > the full power of the fork/join framework through a very simple API based on the streams.

* Two patterns to create parallel streams
  * The first pattern: create the stream by calling parallelStream()
    ```java
    List<Person> people = ...;
    people.parallelStream()
          .filter(person -> person.getAge() > 20)
          .forEach(System.out::println);
    ```

  * The second pattern: call the parallel method on an existing stream
    ```java
    List<Person> people = ...;
    people.stream().parallel()
          .filter(person -> person.getAge() > 20)
          .forEach(System.out::println);
    ```
    > The order in which the people will be printed out is not guaranteed.
    > 
    > To guarantee the order of the elements, we must use sorted() 

* Caveats with Parallel Streams
  * Parallel streams are built on top of the fork/join framewrok, a multithread framework.
    * Some things are to be avoided when computing things with the fork/join
      * Synchronization and visibility issues!
        * A stream should not modify its source.
          * If a stream processing modifies its source and you are going parallel,
            you will go into big troubles.
      * Stateful streams will not be computed efficiently in parallel

### 2. Stateful & stateless operations<a name="stateful-and-stateless-operations"></a>[↑](#top)
* If you have stateful operations in a stream processing, you should not go parallel.
* Stateless operation: A stateless operation is an operation that works on only one element of
  a given stream and does not need any kind of outside information.
  ```java
  List<Person> people = ...;
  people.parallelStream()
        // No outside information is needed to compute this boolean.
        .filter(person -> person.getAge() > 20)
        .forEach(System.out::println);
  ```
  The filter operation in the code above is a stateless operation.
* Stateful operation: 
  ```java
  List<Person> people = ...;
  people.parallelStream()
        .skip(2)
        .limit(5)
        .forEach(System.out::println);
  ```
  The skip and limit operations in the code above are stateful operations.
* How to tell a stateful operation from a stateless one?
  * It is written in the Javadoc
  * Basically, the filtering is stateless, the limit has to be stateful.

* A sneaky stateful operation
  ```java
  List<Person> people = Arrays.asList(p1, p2, p3);
  people.stream().parallel() // this stream is ordered!
        .filter(person -> person.getAge() > 20)
        .forEach(System.out::println)
  ```
  In fact, no operations on the stream above is stateful, what is stateful is the stream itself.
  * How to fix that?
    * Just call the unordered() method to relax the constraint
      ```java
      List<Person> people = Arrays.asList(p1, p2, p3);
      people.stream()
            .unorderded()
            .parallel() // set the ORDERED bit to 0
            .filter(person -> person.getAge() > 20)
            .forEach(System.out::println)
      ```

### 3. Parallel reductions<a name="parallel-reductions"></a>[↑](#top)
* Use collectors instead of reduce()
  * Do not use this code in parallel!
    ```java
    List<Person> people = ...;
    List<Integer> ages =
    people.stream() // no .parallel()
          .reduce(
              new ArrayList<Integer>(),
              (list, p) -> { list.add(p.getAge()); return list; },
              (list1, list2) -> list1
          );
    ```
    > Because ArrayList is not concurrent aware, and race conditions will occur. Different threads
    > will try to add elements to that ArrayList without any synchronization, so it will just not work.
  * Collectors.toList() will handle parallelism and thread-safety for us
    ```java
    List<Person> people = ...;
    List<Integer> ages =
    people.stream.parallel()
          .collect(Collectors.toList());
    ```
* Tuning Parallelism
  * By default, the Fork/Join takes all the available CPUs
    * It uses a specail pool of thread launched when the JVM is started, which is called
      the Common Fork / Join pool of thread.
    * We can control this pool:
      ```java
      System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", 2);
      ```
      This property should be set systematically to 1 with all Java 8 non-streaming operations.
  * And we can also launch our computations in our own pool:
    ```java
    List<Person> people = ...;
    ForkJoinPool fjp = new ForkJoinPool(2);
    fjp.submit(
      () ->                           //
      people.stream().parallel()      // this is an implementation of
            .mapToInt(Person::getAge) // Callable<Integer>
            .filter(age -> age > 20)  // 
            .average()                //
    ).get(); // from Future
    ``` 