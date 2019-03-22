# [技术备忘录](../README.md) | [Java](README.md) | Java8流式数据处理-利用Optionals处理数据处理中的错误
## 目录
  1. [What Optionals are and why they have been introduced on the Stream API?](#what-and-why-of-optionals)
  2. [The first patterns to use Optionals](#first-patterns-to-use-optionals)
  3. [Advanced uses of Optionals](#advanced-uses-of-optionals)
## 问题
### 1. What Optionals are and why they have been introduced on the Stream API?<a name="what-and-why-of-optionals"></a>[↑](#top)

  > When I am not sure that a return value, then I wrap that value in an
  > Optional. The concept of an Optional is there to tell that a value might
  > not be there.
  > 
  > The Optional object can be seen as a wrapper type, the same kind of wrapper
  > type we already have in the JDK with the integer, double, long, etc.

### 2. The first patterns to use Optionals<a name="#first-patterns-to-use-optionals"></a>[↑](#top)
* The first type of patterns to use the Optional class
  > Sees an Optional as a wrapper and I can query that wrapper - Do you have an element?
  > What is the element you're wrapping?
  * A first pattern: use [Optional.isPresent()](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/Optional.html#isPresent()) and [Optional.get()](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/Optional.html#get()):
    ```java
    Optional<Person> opt = ...;
    
    if (opt.isPresent() {
        Person p = opt.get();
    }) else {
        // there is nobody here...
    }
    ```
  * A first variant of this first pattern:
    ```java
    Optional<Person> opt = ...;
    Person p1 = opt.orElse(Person.getDefault());
    ```
    > This variant only works if the notion fo default value is defined in my application.

  * The third pattern: instead of providing a default instance, we provide a way to build
    that default instance:
    ```java
    Optional<Person> opt = ...;
    Person p2 = opt.orElseGet(() -> Person.getDefault());
    ```
    > This pattern is much smarter, and we can use this pattern nearly everywhere in our applications. We can save the overhead of building that instance if it is not needed.

* How to build optionals from scratch?
  * First, the default constructor of the Optional class is private, so we cannot build
    an optional using ```new```.
  * We have static methods:
    ```java
    Optional<String> empty = Optional.empty();
    Optional<String> nonEmpty = Optional.of(s); // NulllPointerException
    Optional<String couldBeEmpty = Optional.ofNullable(s);
    ```
    > So it is impossible to  build optional objects on null object.  

* The second type of patterns to use the Optional class
  > Sees an optional as a special stream, that can hold one or zero element
  ```java
  public Optional<U> map(Function<T, U> mapper);
  ```
  > Returns an empty optional if ```this``` is empty.
  ```java
  public Optional<T> filter(Predicate<T> filter);
  ```
  > Returns an empty optional if ```filter``` returns ```false``` or ```this``` is pty.
  ```java
  public void ifPresent(Consumer<T> consumer);
  ```
  > Does nothing if ```this``` is empty.
  * The map/filter/ifPresent methods of Optional clearly look like map/filter/forEach
    from the Stream interface, so it leads to a question: _is an optional a stream_?

* How to get optionals as results of method calls?

### 3. Advanced uses of Optionals<a name="#advanced-uses-of-optionals"></a>[↑](#top)
* Optionals can be seens as a special type of stream, which would hold only one or zero element. Once we've ssen that we can find more advanced patterns, much more elegant and efficient to compute.

```java
public class NewMath {
    public static Optional<Double> sqrt(Double d) {
        return d >= 0d ? Optional.of(Math.sqrt(d)) : Optional.empty();
    }

    public static Optional<Double> inv(Double d) {
        return d != 0d ? Optional.of(1d/d) : Optional.empty();
    }
}
```
> Below is a data processing pipeline using this NewMath class
```java
List<Double> doubles = ...;
List<Double> result = new ArrayList<>();

doubles.stream()
       .forEach(
           d -> NewMath.sqrt(d)
                       .flatMap(Math::inv)
                       .ifPresent(result::add)
       );
``` 
* What is wrong with this way of processing things?
> There are two problems:
> 
> 1. I'm accumlating the result in an external list, which should be final and I should
>    be the only one to modify it. Since I am the only one to modify it, I cannot go parallel.
> 2. 

* Using [flatMap()](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/Optional.html#flatMap(java.util.function.Function))
  ```java
  Optional<U> flatMap(Function<T, Optional<U>> flatMapper);
  ```

  * Is it possible to build a function that returns a Stream\<T\>?
    > The answer is yes, see the code below:
    ```java
    Function<Double, Stream<Double>> invSqrt =
    d -> NewMath.inv(d)               // Optional<Double>
           .flatMap(NewMath::sqrt)    // Optional<Double>
           .map(Stream::of)           // Optional<Stream<Double>>
           .orElseGet(Stream::empty); // Stream<Double>
    ``` 
  
  * We can then write out data processing pattern
    ```java
    List<Double> doubles = ...;

    List<Double> invSqrtOfDoubles =
    doubles.stream().parallel()
           .flatMap(invSqrt)
           .collect(Collectors.toList()); // collects the elements in a list
    ```
    > And this time we can safely compute this stream in parallel!
    > 
    > In this pattern, I need to handle neither any kind of exception nor any kind of null values, and I get free parallelism.

    * Use parallel on a stream with a source of ArrayList may lead to 
      random ArrayIndexOutOfBoundsException or loss of data.