# [技术备忘录](../README.md) | [Java](README.md) | Java8流式数据处理-高阶主题
## 目录
  1. [What is a Spliterator？](#what-is-spliterator)
  2. [An example of building our own Spliterator](#custom-spliterator-example)

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
  * How the Spliterator is implemented in ArrayList
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
    }
    ```