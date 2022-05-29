## **池化技术（Pool）**

**池化技术 (Pool)** 是一种很常见的编程技巧，我们日常工作中常见的有**数据库连接池、线程池、对象池**等，它们的特点都是将“昂贵的”、“费时的”的资源维护在一个特定的“池子”中，规定其最小连接数、最大连接数、阻塞队列等配置，方便进行统一管理和复用，通常还会附带一些探活机制、强制回收、监控一类的配套功能。

## **线程池（Thread Pool）概念**

顾名思义，线程池可以理解为一个装有线程的池子，这个池可以用来更好的统一管理线程，线程池技术可以帮我们管理线程，避免增加创建线程和销毁线程的资源损耗，我们可以通过线程池重复利用已有的线程，从而避免每次使用的时候都去创建。

## 线程池在 Java 中的抽象

线程池在Java中的抽象是 `Executor` 接口，但是真正的线程池实现其实是 `ThreadPoolExecutor`，该类提供了四个构造用于创建线程池，具体的方法会在下文中详细解释。除此之外，Java 还提供了几种常用的配置好的线程池，通过 `Executors`提供的工厂方法可以快速的创建线程池来使用，这些工厂方法其实也是直接或间接的使用 `ThreadPoolExecutor` 的构造方法来创建的线程池，具体的方法将在下文中详细解释。

## ThreadPoolExecutor

`ThreadPoolExecutor` 是 Java 中对于线程池的具体实现，在 **java.util.concurrent** 包下。其提供了四个构造用于配置和创建线程池，这四个构造方法如下：

```java
/**
 * @param corePoolSize    该线程池中核心线程数最大值
 * @param maximumPoolSize 该线程池中线程总数最大值
 * @param keepAliveTime   非核心线程闲置超时时长
 * @param unit            keepAliveTime 的单位
 * @param workQueue       线程池中的任务队列
 */
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
}

/**
 * @param corePoolSize    该线程池中核心线程数最大值
 * @param maximumPoolSize 该线程池中线程总数最大值
 * @param keepAliveTime   非核心线程闲置超时时长
 * @param unit            keepAliveTime 的单位
 * @param workQueue       线程池中的任务队列
 * @param threadFactory   创建线程的工厂
 */
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory) {
}

/**
 * @param corePoolSize    该线程池中核心线程数最大值
 * @param maximumPoolSize 该线程池中线程总数最大值
 * @param keepAliveTime   非核心线程闲置超时时长
 * @param unit            keepAliveTime 的单位
 * @param workQueue       线程池中的任务队列
 * @param handler         饱和策略
 */
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          RejectedExecutionHandler handler) {
}

/**
 * @param corePoolSize    该线程池中核心线程数最大值
 * @param maximumPoolSize 该线程池中线程总数最大值
 * @param keepAliveTime   非核心线程闲置超时时长
 * @param unit            keepAliveTime 的单位
 * @param workQueue       线程池中的任务队列
 * @param threadFactory   创建线程的工厂
 * @param handler         饱和策略
 */
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
}
```

### 参数介绍：

- **int corePoolSize**

  线程池的核心线程数，默认情况下，核心线程会在线程池中一致存活，即使它们处于闲置状态。如果将 `ThreadPoolExecutor` 的 `allowCoreThreadTimeOut` 属性设置为 `true`，那么闲置的核心线程在等待新任务到来时会有超时策略，这个时间间隔由 `keepAliveTime` 所指定，当等待时间超过 `keepAliveTime` 所指定的时长后，核心线程就会被终止。

- **int maximumPoolSize**

  线程池所能容纳的最大线程数，当活动线程数达到这个数值后，后续的新任务将会被阻塞。

- **long keepAliveTime**

  非核心线程闲置时的超时时长，超过这个时长，非核心线程就会被回收。当 `ThreadPoolExecutor` 的 `allowCoreThreadTimeOut` 属性设置为 `true` 时，`keepAliveTime` 同样会作用于核心线程。

- **TimeUnit unit**

  用于指定 `keepAliveTime` 参数的时间单位，这是一个枚举。

- **BlockingQueue<Runnable> workQueue**

  线程池中的任务队列，通过线程池的 `execute` 方法提交的 `Runnable` 对象会存储在这个队列中。

- **ThreadFactory threadFactory**

  线程工厂，为线程池提供创建新线程的功能。`ThreadFactory` 是一个接口，它只有一个方法：`Thread newThread(Runnable r)`。源码如下：

  ```java
  /**
   * An object that creates new threads on demand.  Using thread factories
   * removes hardwiring of calls to {@link Thread#Thread(Runnable) new Thread},
   * enabling applications to use special thread subclasses, priorities, etc.
   *
   * <p>
   * The simplest implementation of this interface is just:
   *  <pre> {@code
   * class SimpleThreadFactory implements ThreadFactory {
   *   public Thread newThread(Runnable r) {
   *     return new Thread(r);
   *   }
   * }}</pre>
   *
   * The {@link Executors#defaultThreadFactory} method provides a more
   * useful simple implementation, that sets the created thread context
   * to known values before returning it.
   * @since 1.5
   * @author Doug Lea
   */
  public interface ThreadFactory {
  
      /**
       * Constructs a new {@code Thread}.  Implementations may also initialize
       * priority, name, daemon status, {@code ThreadGroup}, etc.
       *
       * @param r a runnable to be executed by new thread instance
       * @return constructed thread, or {@code null} if the request to
       *         create a thread is rejected
       */
      Thread newThread(Runnable r);
  }
  ```

- **RejectedExecutionHandler handler**

  饱和策略。当线程池无法执行新任务时，这可能是由于任务队列已满或者是无法成功执行任务，这个时候 `ThreadPoolExecutor` 会调用 `handler` 的 `void rejectedExecution(Runnable r, ThreadPoolExecutor executor)` 方法来通知调用者，默认情况下 `rejectedExecution()` 方法会直接抛出一个 `RejectedExecutionException` 异常。`RejectedExecutionHandler` 源码如下：

  ```java
  /**
   * A handler for tasks that cannot be executed by a {@link ThreadPoolExecutor}.
   *
   * @since 1.5
   * @author Doug Lea
   */
  public interface RejectedExecutionHandler {
  
      /**
       * Method that may be invoked by a {@link ThreadPoolExecutor} when
       * {@link ThreadPoolExecutor#execute execute} cannot accept a
       * task.  This may occur when no more threads or queue slots are
       * available because their bounds would be exceeded, or upon
       * shutdown of the Executor.
       *
       * <p>In the absence of other alternatives, the method may throw
       * an unchecked {@link RejectedExecutionException}, which will be
       * propagated to the caller of {@code execute}.
       *
       * @param r the runnable task requested to be executed
       * @param executor the executor attempting to execute this task
       * @throws RejectedExecutionException if there is no remedy
       */
      void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
  }
  ```

  `ThreadPoolExecutor` 为 `RejectedExecutionHandler` 提供了四个可选值：`AbortPolicy`、`CallerRunsPolicy`、`DiscardPolicy`、`DiscardOldestPolicy`,其中 `AbortPolicy` 是默认值，它会直接抛出一个 `RejectedExecutionException` 异常。详细解释：

  - **AbortPolicy**

    该策略是默认的拒绝策略，该策略将抛出未检查的 `RejectedExecutionException`，我们可以捕获这个异常，然后根据需求做相应的处理

  - **CallerRunsPolicy**

    直接在 `execute` 方法的调用线程中运行被拒绝的任务，除非执行程序已关闭，在这种情况下任务将被丢弃。

  - **DiscardPolicy**

    它默默地丢弃被拒绝的任务。

  - **DiscardOldestPolicy**

    丢弃最早的未处理请求，然后重试 `execute`。

### **执行任务的规则**

1. 如果线程池中的线程数量未达到核心线程的数量，那么会直接启动一个核心线程来执行任务。
2. 如果线程池中的线程数量已经达到或者超过核心线程的数量，那么任务会被插入到任务队列中排队等待执行。
3. 如果在步骤2中无法将任务插入到任务队列中，这往往是由于队列任务已满，这个时候如果线程数量未达到线程池规定的最大值，那么会立刻启动一个非核心线程来执行任务。
4. 如果步骤3中的线程数量已经达到线程池规定的最大值，那么就拒绝执行此任务， `ThreadPoolExecutor` 会调用 `RejectedExecutionHandler` 的 `rejectedExecution()` 方法来通知调用者。

## **线程池的分类**

上文中提到通过 `Executors`提供的工厂方法可以快速的创建线程池来使用，那么 **JDK 1.8** 的`Executors` 总共可以归纳为提供了6种线程池的创建：

| 工厂方法                                    | 线程池描述            | 线程池具体实现类型                  |
| ------------------------------------------- | --------------------- | ----------------------------------- |
| newFixedThreadPool(int nThreads)            | 线程数固定的线程池    | ThreadPoolExecutor                  |
| newCachedThreadPool()                       | 缓存线程池            | ThreadPoolExecutor                  |
| newScheduledThreadPool(int corePoolSize)    | 定时线程池            | ScheduledThreadPoolExecutor         |
| newSingleThreadExecutor()                   | 单线程线程池          | FinalizableDelegatedExecutorService |
| newSingleThreadScheduledExecutor()          | 单线程定时线程池      | DelegatedScheduledExecutorService   |
| newWorkStealingPool(int parallelism) JDK1.8 | CPU核心数的并行线程池 | ForkJoinPool                        |

- **FixedThreadPool**

  ```java
  public static ExecutorService newFixedThreadPool(int nThreads) {
      return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>());
  }
  
  public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
      return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory);
  }
  ```

  这是一种线程数固定的线程池，当线程处于空闲状态时，它们并不会被回收，除非线程池被关闭了。当所有线程都处于活动状态时，新任务都会处于等待状态，直到有线程空闲出来。由于 `FixedThreadPool` 只有核心线程并且这些核心线程不会被回收，这意味着它能够更加快速的响应外界的请求。另外，任务队列也是没有大小限制的。

- **CachedThreadPool**

  ```java
  public static ExecutorService newCachedThreadPool() {
      return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
  }
  
  public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
      return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>(),
                                    threadFactory);
  }
  ```

  这是一种线程数量不定的线程池，该线程池只有非核心线程，并且最大线程数为 `Integer.MAX_VALUE`。当线程池中的线程都处于活动状态时，线程池会创建新的线程来处理新任务，否则就会利用空闲的线程来处理新任务。线程池中的空闲线程都有超时机制，这个超时时间为60秒，超过该时间闲置线程就会被回收。和`FixedThreadPool` 不同的是，`CachedThreadPool` 的任务队列其实相当于一个空集合，这将导致任何任务都会被立即执行，因为在这种场景下 `SynchronousQueue ` 是无法插入任务的。`SynchronousQueue ` 是一个非常特殊的队列，在很多情况下可以把它简单理解为一个无法存储元素的队列。该线程池比较适合执行大量的耗时较少的任务。当整个线程池都处于空闲状态时，线程池中的线程都会超时而被停止，这个时候 `CachedThreadPool` 之中实际是没有任务线程的，它几乎是不占用任何系统资源的。

- **ScheduledThreadPool**

  ```java
  public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
      return new ScheduledThreadPoolExecutor(corePoolSize);
  }
  
  public static ScheduledExecutorService newScheduledThreadPool(
          int corePoolSize, ThreadFactory threadFactory) {
      return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
  }
  ```

  这是一个核心线程数量固定的，而非核心线程数无限制，并且当非核心线程闲置时会被立即回收的线程池。该线程池主要用于执行定时任务和具有固定周期的重复任务。

- **SingleThreadExecutor**

  ```java
  public static ExecutorService newSingleThreadExecutor() {
      return new FinalizableDelegatedExecutorService
          (new ThreadPoolExecutor(1, 1,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>()));
  }
  
  public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
      return new FinalizableDelegatedExecutorService
          (new ThreadPoolExecutor(1, 1,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(),
                                  threadFactory));
  }
  ```

  这是一个内部只有一个线程的线程池，它确保所有的任务都在同一个线程中按顺序执行。该线程池的意义在于统一所有外界任务到一个线程中，这使得这些任务之间不需要处理线程同步的问题。

- **SingleThreadScheduledExecutor**

  ```java
  public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
      return new DelegatedScheduledExecutorService
          (new ScheduledThreadPoolExecutor(1));
  }
  
  public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory) {
      return new DelegatedScheduledExecutorService
          (new ScheduledThreadPoolExecutor(1, threadFactory));
  }
  ```

  这是一个可以周期性执行任务的单线程线程池。

- **WorkStealingPool**

  ```java
  public static ExecutorService newWorkStealingPool(int parallelism) {
      return new ForkJoinPool
          (parallelism,
           ForkJoinPool.defaultForkJoinWorkerThreadFactory,
           null, true);
  }
  
  public static ExecutorService newWorkStealingPool() {
      return new ForkJoinPool
          (Runtime.getRuntime().availableProcessors(),
           ForkJoinPool.defaultForkJoinWorkerThreadFactory,
           null, true);
  }
  ```

  这是JDK1.8引入的线程池，该线程池使用所有可用处理器作为目标并行度，创建一个窃取线程的池，拥有多个任务队列。其中一个任务可以产生其他较小的任务，这些任务被添加到并行处理线程的队列中，如果一个线程完成了工作并且无事可做，则可以从另一线程的队列中"窃取"工作。这个线程池不会保证任务的顺序执行，因为它的工作方式是抢占式的。该线程池主要使用了 `ForkJoinPool`来作为实现，基于 **work-stealing** 算法。

## **参考：**

[深入Java 8的newWorkStealingPool](https://www.codenong.com/d-diving-into-java-8s-newworkstealingpools/)

《Android 开发艺术探索》