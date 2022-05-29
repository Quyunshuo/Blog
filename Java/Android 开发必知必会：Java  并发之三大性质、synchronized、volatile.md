# 三大性质：原子性、有序性、可见性

> 并发编程中讨论线程安全问题绕不开三大性质：**原子性**、**有序性**、**可见性**

## 原子性

**原子（atomic）** 本意是“不能被进一步分割的最小粒子”，而**原子操作（atomic operation）** 意为“不可被中断的一个或一系列操作”。原子性则可以表示为：一个操作是不可中断的，要么全部执行成功要么全部执行失败，有着“同生共死”的感觉。

## 有序性

指的是在代码顺序结构中，我们可以直观的指定代码的执行顺序, 即从上到下按序执行。但编译器和CPU处理器会根据自己的决策，对代码的执行顺序进行重新排序。优化指令的执行顺序，提升程序的性能和执行速度，使语句执行顺序发生改变，出现重排序，但最终结果看起来没什么变化（单核）。

### 有序性问题

指的是在多线程环境下（多核），由于执行语句重排序后，重排序的这一部分没有一起执行完，就切换到了其它线程，导致的结果与预期不符的问题。这就是编译器的编译优化给并发编程带来的程序有序性问题

### 指令重排序

为了使处理器内部的运算单元能尽量被充分利用，处理器可能会对输入的代码进行乱序执行优化，处理器会在计算之后将乱序执行的结果重组，并确保这一结果和顺序执行结果是一致的，但是这个过程并不保证各个语句计算的先后顺序和输入代码中的顺序一致。这就是指令重排序。

## 可见性

当一个线程修改一个线程共享变量时，另外的线程能够读到这个修改的值。也就是说，被修饰的共享变量被任何线程读取的时候都能拿到最新的值。

# **synchronized**

## **定义：**

> 在多线程的环境下，多个线程同时访问共享资源会出现一些问题，而 **synchronized** 关键字则是用来保证线程同步的。**synchronized** 是 **Java** 提供的一个并发控制的关键字。主要有两种用法，分别是同步方法和同步代码块。也就是说，**synchronized** 既可以修饰方法也可以修饰代码块。
>
> Java 中的每一个对象都可以作为锁，这个对象也被称为 **监视器（monitor）** 。具体表现为以下3种形式：
>
> - 对于普通同步方法，锁是当前实例对象。
> - 对于静态同步方法，琐是当前类的 **Class** 对象。
> -   对于同步方法块，锁是 **Syschonized** 括号里配置的对象。

## **作用：**

给修饰的方法和代码块加锁，保证同时只能有一个线程访问。

## **特点：**

**有序性**、**原子性**、**可见性**

## **使用：**

**Java：**

```java
public class SynchronizedTest {

   private final User user = new User();

   /**
    * 同步方法，监视器为当前对象
    * 此处的锁和 synchronized(this) 是同样的
    */
   public synchronized void synchronizedMethod() {}

   /**
    * 同步静态方法，监视器为当前类的 Class 对象
    * 此处的锁和 synchronized(SynchronizedTest.class) 是同样的
    */
   public synchronized static void synchronizedStaticMethod() {}

   /**
    * 同步代码块，监视器为 synchronized(object) 传入的对象
    */
   public void synchronizedCodeBlock() {

       /* 监视器为 user 对象，同时只能有一个线程拿到 user 锁 */
       synchronized (user) {
           System.out.println(user.name);
      }

       /* 监视器为当前类的实例对象，同时只能有一个线程拿到该类的实例锁 */
       synchronized (this) {
           System.out.println("SynchronizedTest");
      }

       /* 监视器为当前类的 Class 对象，同时只能有一个线程拿到当前类的 Class 对象锁 */
       synchronized (SynchronizedTest.class) {
           System.out.println("SynchronizedTest.class");
      }
  }
}
```

**Kotlin：**

```kotlin
class SynchronizedTestKt {

    private val user = User()

    /**
     * 同步方法，监视器为当前对象
     * 此处的锁和 synchronized(this) 是同样的
     */
    @Synchronized
    fun synchronizedMethod() {}

    /**
     * 同步代码块，监视器为 synchronized(object) 传入的对象
     */
    fun synchronizedCodeBlock() {

        /* 监视器为 user 对象，同时只能有一个线程拿到 user 锁 */
        synchronized(user) { println(user.name) }

        /* 监视器为当前类的实例对象，同时只能有一个线程拿到该类的实例锁 */
        synchronized(this) { println("SynchronizedTestKt") }

        /* 监视器为当前类的 Class 对象，同时只能有一个线程拿到当前类的 Class 对象锁 */
        synchronized(SynchronizedTestKt::class.java) { println("SynchronizedTestKt.class") }
    }

    /**
     * 伴生对象
     */
    companion object {

        /**
         * 同步伴生对象方法，监视器为当前类的 Class 对象
         * 此处的锁和 synchronized(SynchronizedTestKt::class.java) 是同样的
         */
        @Synchronized
        fun synchronizedStaticMethod() {}
    }
}
```

# **volatile**

## **定义：**

> Java 语言规范第3版中对 **volatile** 的定义如下：**Java 编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量。**
>
> Java 语言提供了 **volatile**，在某些情况下比锁要更加方便。如果一个字段被声明成 **volatile**，**Java 线程内存模型**确保所有线程看到这个变量的值是一致的 。**volatile** 是一种轻量且在有限的条件下线程安全技术，它保证修饰的变量的可见性和有序性，但非原子性。相对于 **synchronize** 高效，而常常跟 **synchronize** 配合使用。

## **作用：**

1.  保证了不同线程对该变量操作的内存可见性
1.  禁止指令重排序

## **特点：**

**有序性**、**非原子性**、**可见性**

## **实现原理：**

**引《Java 并发编程的艺术》书中的例子：**

在 **X86** 处理器下通过工具获取 **JIT** 编译器生成的汇编指令来查看对 **volatile** 进行写操作时，**CPU** 会做什么事。

**Java 代码：**

```java
instance = new Singleton;    // instance 是被 volatile 修饰的变量
```

**转为汇编：**

```
0x01a3de1d: movb $0×0,0×1104800(%esi);0x01a3de24: lock addl $0×0,(%esp);
```

有 **volatile** 变量修饰的共享变量进行写操作的时候会多出第二行汇编代码，通过查 **IA-32架构软件开发者手册**可知，**Lock** 前缀的指令在多核处理器下会引发了两件事情：

1.  将当前处理器缓存行的数据写回到系统内存。
1.  这个写回内存的操作会使在其他CPU里缓存了该内存地址的数据无效。

为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存(L1，L2或其他)后再进行操作，但操作完不知道何时会写到内存。如果对声明了 **volatile** 的变量进行写操作，**JVM** 就会向处理器发送一条 **Lock** 前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

**volatile** 的两条实现原则：

1.  **Lock** 前缀指令会引起处理器缓存回写到内存。
1.  一个处理器的缓存回写到内存会导致其他处理器的缓存无效。

## **使用：**

**Java：**

```java
public class Main {

   private volatile int variable = 0;
}
```

**Kotlin：**

```kotlin
class Main {

   @Volatile
   private var variable: Int = 0
}
```

# **参考：**

[面试官最爱的volatile关键字](https://juejin.cn/post/6844903520760496141)

《Java 并发编程的艺术》