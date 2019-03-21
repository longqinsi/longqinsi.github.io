# [技术备忘录](../README.md) | [Java](README.md) | Java8流式数据处理-高阶主题
## 目录
  1. [What is a Spliterator？](#what-is-spliterator)
  2. [An example of building our own Spliterator](#custom-spliterator-example)
  3. [Merge Streams](#merge-streams)
  4. [State of a Stream](#state-of-a-stream)
  5. [Stream of numbers](#stream-of-numbers)
## 问题
### 1. What is a [Spliterator](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/Spliterator.html)？<a name="what-is-spliterator"></a>[↑](#top)
It is a special object on which a Stream is built.

It is a special object used to connect a stream to a nonstandard source.

It is a new interface in Java 8, that models the access to the data for a Stream.
> The collection access its data through the iterator object and the
> stream does the same through the spliterator object.

* A Stream is divided into two things:
  * An object to access the data, this is the Spliterator
    * It is meant to be overriden to suit our needs
  * An object to handle the processing of the data, this is the ReferencePipeline
    > ReferencePipeline is in fact the implementation of the Stream interface that
    > holds all the algorithm and especially the map/filter/reduce algorithm.
    > 
    > _It is a very complex object, we need not to override it._

### 2. An example of building our own Spliterator<a name="custom-spliterator-example"></a>[↑](#top)
The definition of Spliterator：
```java
public interface Spliterator<T> {
    boolean tryAdvance(Consumer<? super T> action);
    Spliterator<T> trySplit();
    long estimateSize();
    int charateristics();

    default void forEachRemaining(Consumer<? super T> action) {
        do { } while (tryAdvance(action));
    }

    default long getExactSizeIfKnown() {
        return (characteristics() & SIZED) == 0 ? -1L : estimateSize();
    }

    default boolean hasCharacteristics(int charateristics) {
        return (characteristics() & characteristics) == characteristics;
    }

    public static final int ORDERED    = 0x00000010;
    public static final int DISTINCT   = 0x00000001;
    public static final int SORTED     = 0x00000004;
    public static final int SIZED      = 0x00000040;
    public static final int NONNULL    = 0x00000100;
    public static final int IMMUTABLE  = 0x00000400;
    public static final int CONCURRENT = 0x00001000;
    public static final int SUBSIZED   = 0x00004000;
    
}
```

* The ArrayList Example
  * How the Spliterator is implemented in ArrayList<a name="array-list-spliterator"></a>
    ```java
    static final class ArrayListSpliterator<E> implements Spliterator<E> {

        private final ArrayList<E> list;
        private int index; // current index, modified on advance/split
        private int fence; // -1 until used; then one past last index
        private int expectedModCount; // initialized when fence set

        public long estimateSize(){
            return (long)(getFence() - index);
        }

        public boolean tryAdvance(Consumer<? super E> action) {
            int hi = getFence(), i = index;
            if (i < hi) {
                index = i + 1;
                E e = (E)list.elementData[i];
                action.accept(e);
                return true;
            }
            return false;
        }

        public ArrayListSpliterator<E> trySplit() {
            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
            return (lo >= mid) ? null : // divide range in half unless too small
                new ArrayListSpliterator<E>(list, lo, index = mid, expectedModCount);
        }

        public int characteristics() {
            return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
        }
    }
    ```

### 3. Merge Streams<a name="merge-streams"></a>[↑](#top)

* The first pattern - [Concat](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#concat(java.util.stream.Stream,java.util.stream.Stream))
  * We can open a stream on the lines of a text file
    ```java
    Path path = Paths.get("files/text1.txt");
    try (Stream<String> s1 = Files.lines(path)) { // Stream is autocloseable
    
        // handle the file line by line
    
    } catch (Exception e) {
        // handle the exception
    }
    ```
  
  * And then concatenate the two streams using [Stream.concat](https://  docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/  Stream.html#concat(java.util.stream.Stream,java.util.stream.Stream))
    ```java
    Stream<String> s1 = Files.lines(path1);
    Stream<String> s2 = Files.lines(path2);
  
    Stream<String> s10 = Stream.concat(s1, s2); // only two
    ```
    > The first problem with concat is that it supports concatenation of only two   streams.
    > If you want to concatenation more than two streams, you need nested concat   calls such as ```Stream<String> s = Stream.concat(Stream.concat(s1, s2), s3);```,
    > then you may run into the risk of a StackOverflow exception or things like   that.
    >
    > The second problem is that in the stream produced by concat method, the   elements
    > are followed by all the elements of the second stream, it is **not the mixing**
    > of the two stream, by the strict **concatenation** of the two stream.
    > 
    > So the order of the elements is preserved, which has a cost.
    > If it is not needed, then we should use the other pattern.

* The second pattern: Stream of Streams
  ```java
  Stream<Stream<String>> s = Stream.of(s1, s2, s3);
  ```
  * We can use [```<R> Stream<R> flatMap​(Function<? super T,​? extends Stream<? extends R>> mapper)```](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#flatMap(java.util.function.Function)) to flatten the Stream of Stream.
    * The mapper parameter is a function that has to return a stream
      * Special case: if it can be an identity function, then we can use it with
        flatMap to _flatten_ a stream of streams.
        ```java
        Function<Stream<String>, Stream<String>> idFlatMapper = stream -> stream;
        Stream<Stream<String>> streamOfStreams = Stream.of(s1, s2, s3);
        Stream<String> stream = s.flatMap(idFlatMapper);
        ```
        The code above can be shortten to as below:
        ```java
        Stream<String> stream = Stream.of(s1, s2, s3)
                                      .flatMap(Function.indentity());
        ```

  * Other flatMap() Use Cases
    * The sample code above merge the lines of two files into one stream.
    * We can also split those lines into words, and merge them into one stream
      of words.
      ```java
      Function<String, Stream<String>> splitIntoWords =
        line -> Pattern.compile(" ").splitAsStream(line);
      // extract all the words from all the files we provide, no duplicate words.
      Set<String> words =
        Stream.of(s1, s2, s3)           // stream of streams of lines
          .flatMap(Function.identity()) // stream of lines
          .flatMap(splitIntoWords)      // stream of words
          .collect(Collectors.toSet());
      ```

### 4. State of a Stream<a name="state-of-a-stream"></a>[↑](#top)
* The stream does not hold data, but it has a state.
  * See [the previous ArrayListSpliterator code example](#array-list-spliterator) 

Characteristic | --
-------------- | --
ORDERED | The order matters
DISTINCT | No duplication
SORTED | Sorted
SIZED | The size is known
NONNULL | No null values
IMMUTABLE | Immutable
CONCURRENT | Parallelism is possible
SUBSIZED | The size is known

> SUBSIZED - Characteristic value signifying that all Spliterators resulting from trySplit() will be both SIZED and SUBSIZED. (This means that all child Spliterators, whether direct or indirect, will be SIZED.)
> 
> A Spliterator that reports SORTED must also report ORDERED.

Method | Set to 0 | Set to 1
------ | -------- | --------
filter() | SIZED | -
map() | DISTINCT, SORTED | -
flatMap() | DISTINCT, SORTED, SIZED | -
sorted() | - | SORTED, ORDERED
distinct() | - | DISTINCT
limit() | SIZED | -
peek() | - | -
unordered() | ORDERED, SORTED | -

> The sorted() method can only be called on a stream of Comparable objects.
> But is is not verified at compile time, so if my stream is not a stream
> of comparable objects, calling sorted() on it will generate a ClassCastException
> which says that the class parameter of the stream cannot be cast to class
> java.lang.Comparable.
> 
> sorted() also has an overload that takes a Comparator as a parameter, that will
> be used to compare the objects of the stream, see [```Stream<T> sorted​(Comparator<? super T> comparator)```](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Stream.html#sorted(java.util.Comparator)).

### 5. Stream of numbers<a name="stream-of-numbers"></a>[↑](#top)
* IntStream, LongStream, DoubleStream
```java
List<Person> people = ...;
Optional<Double> averageAge =
  people.stream()                      // Stream<Person>
        .mapToInt(Person::getAge)      // IntStream
        .filter(age -> age > 20)
        .average();
```
* Streams of numbers are there to avoid the cost of boxing/unboxing
* The patterns to build them:
  ```java
  // build from a varargs
  LongStream streamOfLongs = LongStream.of(1L, 2L, 3L);
  ```

  ```java
  // convert from a Stream<Person>
  IntStream streamOfInts = people.stream().mapToInt(Person::getAge);
  ```
* The pattern to box a Stream of numbers:
  ```java
  // box a Stream if needed
  Stream<Long> boxedStream = LongStream.of(1L, 2L, 3L).boxed();

  // or use mapToObj method
  boxedStream = LongStream.of(1L, 2L, 3L).mapToObj(l -> l);
  ```
* Stream of numbers have special methods, not on Stream\<T\>
  * IntStream
    ```java
    int sum = intStream.sum();

    OptionalInt min = intStream.min();
    OptionalInt max = intStream.max();

    OptionalDouble average = intStream.average();

    IntSummaryStatistics stats = intStream.summaryStatistics();
    ```
    > Summary statistics compute sum, min, max, count and average in one pass.
