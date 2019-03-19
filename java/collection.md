# [技术备忘录](../README.md) | [Java](README.md) | 集合

## 目录
  1. [Java 8为集合新增了哪些方法？](#java8-new-collection-api)

## 问题
### 1.Java 8为集合新增了哪些方法？<a name="java8-new-collection-api"></a>[↑](#top)
#### 1) Iterable
* boolean forEach(Consumer<? super E> consumer);

例子：
```java
List<Person> people = ... ;
// 把列表中所有人的信息输出到控制台
people.forEach(System.out::println);
```
#### 2) Collection
* boolean removeIf(Predicate<? super E> filter);

例子：
```java
List<Person> people = ... ;
// 从列表中移除所有年龄小于20的人
people.removeIf(person -> person.getAge() < 20);
```

#### 3) List
* boolean replaceAll(UnaryOperator<? super E> operator);

例子：
```java
List<String> names = ... ;
// 把列表中所有人的姓名转为大写
names.replaceAll(String::toUpperCase);
```
* boolean sort(Comparator<? super E> comparator);

例子：
```java
List<Person> people = ... ;
// 把列表中的人首先按姓名，然后按年龄排序
names.sort(
    Comparator.comparing(Person::getName)
        .thenComparing(Person::getAge)
);
```

#### 4) Map

* void forEach(BiConsumer<? super K, ? super V> consumer);

例子:
```java
Map<City, List<Person>> map = ... ;
// 把字典中的每个城市及其包含的人数输出到控制台
map.forEach(
    (city, list) ->
        System.out.println(city + ": " + list.size() + " people")
);
```

* V getOrDefault(Object key, V defaultValue);
  * Allows one to check if a key is present in a map or not

例子
```java
Map<City, List<Person>> map = ... ;
// 打印字典中波士顿对应的人的列表；如果字典中不包含波士顿，则打印一个空列表。
System.out.println(map.getOrDefault(boston, emptyList());
```

* V putIfAbsent(K key, V value);

例子
```java
Map<City, List<Person>> map = ... ;
// 如果列表中没有波士顿，则为其初始化一个空列表放入字典中，并将这个空列表作为方法返回值；
// 否则不对字典做任何变化，把字典中原来包含的波士顿对应的列表作为方法返回值
map.putIfAbsent(boston, new ArrayList<>());
// 在向值为集合的字典中的某个键对应的集合添加元素之前，用上一行的模式可以避免NullPointerException。
map.get(boston).add(maria);
```

* V replace(K key, V newValue);
* boolean replace(K key, V existingValue, V newValue);
  * 只有 key 在字典里，并且 key 的对应值与 existingValue 相等，才会用 newValue 替换 key 的对应值。如果发生了替换，返回true；否则返回false。
* void replaceAll(Bifunction<? super K, ? super V, ? extends V> function);
  * 对字典中所有键值对执行function，用根据原有键/值对生成的新值替换原有的值。

* boolean remove(Object key, Object value);
  * 只有 key 在字典里，并且 key 的对应值与 value 相等，才会从字典中移除key对应的键值对。如果发生了移除，返回true；否则返回false。

* compute*
  * V compute(K key, Bifunction<? super K, ? super V, ? extends V> remapping);
    * 如果 key 在字典里，把 key 和字典中 key 对应的值分别作为第一个和第二个参数调用 remapping，计算得到的值作为   compute 方法的返回值
    * 如果 key 不在字典里，把 key 和 ```null``` 分别作为第一个和第二个参数调用remapping，计算得到的值作为compute方  法的返回值。
  * V computeIfAbsent(K key, Function<? super K,? extends V> mapping);
    * 如果 key 在字典里，把字典key对应的值作为 computeIfAbsent 方法返回值
    * 如果 key 不在字典里，把 key 作为参数调用mapping，计算得到的值作为 computeIfAbsent 方法的返回值。
  * V computeIfPresent(K key, BiFunction<? super K,? super V, ? extends V> remapping);
    * 如果 key 在字典里，把 key 和对应值分别作为第一个和第二个参数调用 remapping，计算得到的值作为 compute 方法的返回值
    * 如果 key 不在字典里，返回 ```null``` 。
  * 例子：

    1. 构建嵌套字典（字典的字典）
      ```java
      Map<String, Map<String, Person>> map = ...;
      // key, newValue
      map.computeIfAbsent(
          "one",
          key -> new HashMap<String, Person>()
      ).put("two", john);
      ```

    2. 构建列表的字典（字典的值是列表）
      ```java
      Map<String, List<Person>> map = ...;
      // key, newValue
      map.computeIfAbsent(
          "one",
          key -> new ArrayList<Person>()
      ).add(john);
      ```

* merge 方法，用于合并字典
  * V merge(K key, V newValue, BiFunction<? super V,? super V, ? extends V> remapping);
    * 如果 key 不在字典里，或对应值为 ```null```，把 key/newValue 作为键值对加入字典。
    * 否则，把 key 的对应值和 newValue 分别作为第一个和第二个参数调用 remapping 函数，根据计算结果：
      * 如果计算结果不是```null```，把计算结果作为 key 新的对应值
      * 否则，从字典中移除 key
    * 返回值：字典中 key 在调用 merge 方法后最终的对应值；如果在调用 merge 方法后，字典中不包含 key，
      则返回```null```。
    * 例子：下面的代码把 map2 的所有元素合并到 map1 中
    ```java
    Map<City, List<Person>> map1 = new HashMap<>();
    Map<City, List<Person>> map2 = new HashMap<>();
    map2.forEach(
        (key, value) ->
            map1.merge(
                key, value,
                (existingPeople, newPeople) -> {
                    existingPeople.addAll(newPeople);
                    return existingPeople;
                }
            )
    );
    ```

