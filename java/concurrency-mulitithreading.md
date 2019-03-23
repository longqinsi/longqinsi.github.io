# [技术备忘录](../README.md) | [Java](README.md) | 并发和多线程

## 目录
  1. [What is concurrency？](#what-is-concurrency)

## 问题
### 1. What is a Thread?
* A thread is defined at the Operating System level
  * From the developer point of view, a thread is a set of instructions that I'm going to write in my application, and execute in a certain way.
  * An application can be composed of several threads.
  * Different threads can be executed "at the same time"
  * The Java Virtual Machine works with several threads (GC, JIT, ...)
* The most basic way to create threads in Java is to use the Runnable Pattern.
  1) Create an instance of Runnable
  2) Pass it to the constructor of the Thread class
  3) Call the start() method of this Thread object
  ```java
  Runnable runnable = () -> {
    String name = Thread.currentThread().getName();
    System.out.println("I am running in thread " + name);
  }
  Thread thread = new Thread(runnable);
  thread.start();
  ```
  The code above can be written in one line:
  ```java
  new Thread(() -> {
    String name = Thread.currentThread().getName();
    System.out.println("I am running in thread " + name);
  }).start();
  ```  

### 1. What is concurrency？<a name="what-is-concurrency"></a>[↑](#top)
* Concurrency: "The art of doing several things at the **same** time"
  * What does the word "same" mean in this context?
    > This context has changed from the monocore CPUs to the multicore CPUs.
    > 
    > In the monocore CPU era, we feel things happen at the same time because things are happening fast, but in CPU level, things are in fact executed sequentially.
    > 
    > In the multicore CPU ear, things are really happening at the same time.

### 2. When will the scheduler pause a thread?
1) The CPU should be shared equally among threads
2) The thread is waiting for some more data
3) The thread is waiting for another thread to do something(e.g. to release a resource)

### 2. What does correct code mean in the concurrent world? 

### 3. How to improve your code by leveraging multi-core CPUS?
* How we can improve existing code or new code by leveraging multi-core CPUs - improving the performance of our code.
  * See how to write code, how to implement patterns - esp. the singleton pattern.

### 4. Race condition, synchronization, volatility
> Race condition deals with the access of data concurrently.
  * What does it mean accessing data concurrently?
    > It means that two different threads might be reading the same variable,
    > the same field defined in a Java class, or the same array.
  * Accessing data concurrently may lead to issues
  * Race condition means that two different threads are trying to read and write the same variable at the same time.
    * An example: The problem with a singleton pattern implementation in the concurrent world:
      ```java
      public class Singleton {
        private static Singleton instance;
        private Singleton() {}
        public static Singleton getInstance() {
          if (instance == null) {
            instance = new Singleton();
          }
          return instance;
        }
      }
      ```
      > What is happening if two threads are calling getInstance() at the same time?
      ![](/images/singleton-race-condition.png)
      > Thread T1 and Thread T2 will both create an Singleton instance respectively, and T1 will erase the instance created by T2
  * How to solve the problem caused by race condition?
    * The answer is: Synchronization
      * Synchronization prevents a block of code to be executed by more than one thread at the same time
        > From a technical point of view, it will prevent the thread scheduler to give the hand to a thread that wants to execute the synchronized portion of code that is being executed by another thread.
      * How does it work technically?
        * very simple - just add the synchronized keyword on the declaration of the method
          ```java
          public class Singleton {
            private static Singleton instance;
            private Singleton() {}
            public static synchronized Singleton getInstance() {
              if (instance == null) {
                instance = new Singleton();
              }
              return instance;
            }
          }
          ```
          > For synchronization to work, we need a specail, technical object that will hold the key.
          > 
          > In fact, every Java object can play this role. 
          > s
          > This key is also called a monitor.

        * How can we deignate the object used to hold the key for synchronization?
          > There are several cases to consider:
          * A synchronized static method uses the class as a synchronization object.
          * A synchronized non-static method uses the instance as a synchronization object
          * A third possibility is to use a dedicated object to synchronize.
            ```java
            public class Person {
              private final Object key = new Object();
              public String init() {
                synchronized(key) {
                  // do some stuff
                }
              }
            }
            ```
            > It is always a good idea to hide an object used for synchronization, whether we are in a static context or not.

  * Reentrant locks
    * In Java, locks are reentrant.
      > When a thread holds a lock, it can enter a block synchronized on the lock it is holding
  * Deadlock
    * A deadlock is a situation where a thread T1 holds a key needed by a thread T2, and T2 holds a key needed by T1.
      > The JVM is able to detect deadlock situations, and can log information to help debug the application. 


### 5. Visibility, false sharing, happens-before link

--------------------------------------------------

### 6. How to implement the producer/consumer pattern using wait/notify?

### 7. Ordering reads and writes operations on a multicore CPU
> Concurrent programming has nothing to do on a monocore CPU and on a multicore CPU.
>  
> Ordering reads and writes operations is the biggest problem on multicore CPUs.

### 8. Implementing a thread safe singleton on a multicore CPU
> This is an application, a use case description, implementing a thread safe version of a singleton pattern on a multicore CPU. It is the occasion to show an example and to come back on all the abstract concepts we are going to describe in this course.