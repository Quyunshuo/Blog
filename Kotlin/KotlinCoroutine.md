# Kotlin Coroutine 简介

## Kotlin Coroutine Version

> **Kotlin Version**: 1.4.32
>
> **Coroutine Version**: 1.4.3

## Kotlin Coroutine 生态

![协程生态.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af760b94ac4f4cab879d55cad89058a6~tplv-k3u1fbpfcp-watermark.image)

## 接入Coroutine

```groovy
dependencies {
    // Kotlin
    implementation "org.jetbrains.kotlin:kotlin-stdlib:1.4.32"

    // 协程核心库
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.4.3"
    // 协程Android支持库
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.4.3"
    // 协程Java8支持库
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-jdk8:1.4.3"
}
```

# Coroutine 基本使用

## 创建协程

# Coroutine的深入

> ​		其实简单的使用，就已经满足大部分日常开发需求，但是我们有必要全面了解一下`Coroutine`，以便能够排查问题及自定义场景，下面我们从一个最基本的函数来切入，这个函数就是`launch{}`：

```kotlin
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,	// 协程上下文
    start: CoroutineStart = CoroutineStart.DEFAULT,			// 协程启动模式
    block: suspend CoroutineScope.() -> Unit						// 运行在协程的逻辑
): Job {
    val newContext = newCoroutineContext(context)
    val coroutine = if (start.isLazy)
        LazyStandaloneCoroutine(newContext, block) else
        StandaloneCoroutine(newContext, active = true)
    coroutine.start(start, coroutine, block)
    return coroutine
}
```

上面是`launch`函数的定义，它以`CoroutineScope`的扩展函数的形成出现，函数参数分别是：`协程上下文CoroutineContext`、`协程启动模式CoroutineStart`、`协程体`，返回值是`协程实例Job`，其中`CoroutineContext`又包括了`Job`、`CoroutineDispatcher`、`CoroutineName`。下面我们就一一介绍这些内容：`CoroutineContext`、`Job`、`CoroutineDispatcher`、`CoroutineStart`、`CoroutineScope`。

## CoroutineContext - 协程上下文

> ​		`CoroutineContext`即协程的上下文，是 Kotlin 协程的一个基本结构单元。巧妙的运用协程上下文是至关重要的，以此来实现正确的线程行为、生命周期、异常以及调试。它包含用户定义的一些数据集合，这些数据与协程密切相关。它是一个有索引的 `Element` 实例集合。这个有索引的集合类似于一个介于 `set` 和 map之间的数据结构。每个 `element` 在这个集合有一个唯一的 Key 。当多个 `element` 的 key 的引用相同，则代表属于集合里同一个 `element`。`CoroutineContext` 有两个非常重要的元素 — `Job` 和 `Dispatcher`，`Job` 是当前的 `Coroutine` 实例而 Dispatcher 决定了当前 `Coroutine` 执行的线程，还有一个元素是`CoroutineName`，用于调试，它们三者都实现了`Element`接口。看一个例子：

```kotlin
fun main() {
    val coroutineContext = Job() + Dispatchers.Default + CoroutineName("myContext")
    println("$coroutineContext,${coroutineContext[CoroutineName]}")
    val newCoroutineContext = coroutineContext.minusKey(CoroutineName)
    println("$newCoroutineContext")
}
```

输出结果如下：

```
[JobImpl{Active}@7eda2dbb, CoroutineName(myContext), Dispatchers.Default],CoroutineName(myContext)
[JobImpl{Active}@7eda2dbb, Dispatchers.Default]
```

`CoroutineContext`接口的定义如下：

```kotlin
public interface CoroutineContext {
    
    public operator fun <E : Element> get(key: Key<E>): E?

    public fun <R> fold(initial: R, operation: (R, Element) -> R): R

    public operator fun plus(context: CoroutineContext): CoroutineContext{...}

    public fun minusKey(key: Key<*>): CoroutineContext

    public interface Key<E : Element>

    public interface Element : CoroutineContext {...}
}
```

`CoroutineContext` 定义了四个核心的操作：

* **操作符get**

  可以通过 `key` 来获取这个 `Element`。由于这是一个 `get` 操作符，所以可以像访问 map 中的元素一样使用 `context[key]` 这种中括号的形式来访问。

* **操作符 plus**

  和 `Set.plus` 扩展函数类似，返回一个新的 `context` 对象，新的对象里面包含了两个里面的所有 `Element`，如果遇到重复的（Key 一样的），那么用`+`号右边的 `Element` 替代左边的。`+`  运算符可以很容易的用于结合上下文，但是有一个很重要的事情需要小心 —— 要注意它们结合的次序，因为这个  `+` 运算符是不对称的。

* **fun <R> fold(initial: R, operation: (R, Element) -> R): R**

  和 `Collection.fold` 扩展函数类似，提供遍历当前 `context` 中所有 `Element` 的能力。

* **fun minusKey(key: Key<*>): CoroutineContext**

  返回一个上下文，其中包含该上下文中的元素，但不包含具有指定`key`的元素。

某些情况需要一个上下文不持有任何元素，此时就可以使用 `EmptyCoroutineContext` 对象。可以预见，添加这个对象到另一个上下文不会对其有任何影响。

## Job & Deferred - 任务

> ​		`CoroutineScope.launch` 函数返回的是一个 `Job` 对象，代表一个异步的任务。`Job` 具有生命周期并且可以取消。 `Job` 还可以有层级关系，一个`Job`可以包含多个子`Job`，当父`Job`被取消后，所有的子`Job`也会被自动取消；当子`Job`被取消或者出现异常后父`Job`也会被取消。
>
> ​		除了通过 `CoroutineScope.launch` 来创建`Job`对象之外，还可以通过 `Job()` 工厂方法来创建该对象。默认情况下，子`Job`的失败将会导致父`Job`被取消，这种默认的行为可以通过 `SupervisorJob` 来修改。
>
> ​		具有多个子 `Job` 的父`Job` 会等待所有子`Job`完成(或者取消)后，自己才会执行完成

### Job 的状态

| **State**                        | [isActive] | [isCompleted] | [isCancelled] |
| -------------------------------- | ---------- | ------------- | ------------- |
| _New_ (optional initial state)   | `false`    | `false`       | `false`       |
| _Active_ (default initial state) | `true`     | `false`       | `false`       |
| _Completing_ (transient state)   | `true`     | `false`       | `false`       |
| _Cancelling_ (transient state)   | `false`    | `false`       | `true`        |
| _Cancelled_ (final state)        | `false`    | `true`        | `true`        |
| _Completed_ (final state)        | `false`    | `true`        | `false`       |

```
                                      wait children
+-----+ start  +--------+ complete   +-------------+  finish  +-----------+
| New | -----> | Active | ---------> | Completing  | -------> | Completed |
+-----+        +--------+            +-------------+          +-----------+
                 |  cancel / fail       |
                 |     +----------------+
                 |     |
                 V     V
             +------------+                           finish  +-----------+
             | Cancelling | --------------------------------> | Cancelled |
             +------------+                                   +-----------+
```

关于`Job`的状态描述，在`kotlinx.coroutines` 库`Job.kt`的注释中描述的非常清楚

### Job 的常用函数

> 这些函数都是线程安全的，所以可以直接在其他 `Coroutine` 中调用。

- **fun start(): Boolean**

  调用该函数来启动这个 `Coroutine`，如果当前 `Coroutine` 还没有执行调用该函数返回 `true`，如果当前 `Coroutine` 已经执行或者已经执行完毕，则调用该函数返回 `false`

- **fun cancel(cause: CancellationException? = null)**

  通过可选的取消原因取消此作业。 原因可以用于指定错误消息或提供有关取消原因的其他详细信息，以进行调试。

- **fun invokeOnCompletion(handler: CompletionHandler): DisposableHandle** 

  通过这个函数可以给 `Job` 设置一个完成通知，当 `Job` 执行完成的时候会**同步**执行这个通知函数。 回调的通知对象类型为：`typealias CompletionHandler = (cause: Throwable?) -> Unit`.

  `CompletionHandler` 参数代表了 `Job` 是如何执行完成的。 `cause` 有下面三种情况：
  – 如果 `Job` 是正常执行完成的，则 `cause` 参数为 `null`
  – 如果 `Job` 是正常取消的，则 `cause` 参数为 `CancellationException` 对象。这种情况不应该当做错误处理，这是任务正常取消的情形。所以一般不需要在错误日志中记录这种情况。
  – 其他情况表示 `Job` 执行失败了。

  这个函数的返回值为 `DisposableHandle` 对象，如果不再需要监控 `Job` 的完成情况了， 则可以调用 `DisposableHandle.dispose` 函数来取消监听。如果 `Job` 已经执行完了， 则无需调用 `dispose` 函数了，会自动取消监听。

- **suspend fun join()**

  `join` 函数和前面三个函数不同，这是一个 `suspend` 函数。所以只能在 Coroutine 内调用。

  这个函数会暂停当前所处的 `Coroutine`直到该`Coroutine`执行完成。所以 `Job` 函数一般用来在另外一个 `Coroutine` 中等待 job 执行完成后继续执行。当 `Job` 执行完成后， `job.join` 函数恢复，这个时候 `job` 这个任务已经处于完成状态了，而调用 `job.join` 的Coroutine还继续处于 `activie` 状态。

  请注意，只有在其所有子级都完成后，作业才能完成

  该函数的挂起是可以被取消的，并且始终检查调用的`Coroutine`的`Job`是否取消。如果在调用此挂起函数或将其挂起时，调用`Coroutine`的`Job`被取消或完成，则此函数将引发 `CancellationException`。

### Deferred

```kotlin
public interface Deferred<out T> : Job {

    public val onAwait: SelectClause1<T>

    public suspend fun await(): T

    @ExperimentalCoroutinesApi
    public fun getCompleted(): T

    @ExperimentalCoroutinesApi
    public fun getCompletionExceptionOrNull(): Throwable?
}
```



> ​		通过使用`async`创建协程可以得到一个有返回值`Deferred`，`Deferred` 接口继承自 `Job` 接口，额外提供了获取 `Coroutine` 返回结果的方法。由于 `Deferred` 继承自 Job 接口，所以 `Job` 相关的内容在 `Deferred` 上也是适用的。 Deferred 提供了额外三个函数来处理和`Coroutine`执行结果相关的操作。

* **suspend fun await(): T**

  用来等待这个`Coroutine`执行完毕并返回结果。

* **fun getCompleted(): T**

  用来获取`Coroutine`执行的结果。如果`Coroutine`还没有执行完成则会抛出 IllegalStateException ，如果任务被取消了也会抛出对应的异常。所以在执行这个函数之前，可以通过 `isCompleted` 来判断一下当前任务是否执行完毕了。

* **fun getCompletionExceptionOrNull(): Throwable?**

  获取已完成状态的`Coroutine`异常信息，如果任务正常执行完成了，则不存在异常信息，返回null。如果还没有处于已完成状态，则调用该函数同样会抛出 `IllegalStateException`，可以通过 `isCompleted` 来判断一下当前任务是否执行完毕了。

### SupervisorJob

`SupervisorJob` 是一个顶层函数，定义如下：

```kotlin
/**
 * Creates a _supervisor_ job object in an active state.
 * Children of a supervisor job can fail independently of each other.
 * 
 * A failure or cancellation of a child does not cause the supervisor job to fail and does not affect its other children,
 * so a supervisor can implement a custom policy for handling failures of its children:
 *
 * * A failure of a child job that was created using [launch][CoroutineScope.launch] can be handled via [CoroutineExceptionHandler] in the context.
 * * A failure of a child job that was created using [async][CoroutineScope.async] can be handled via [Deferred.await] on the resulting deferred value.
 *
 * If [parent] job is specified, then this supervisor job becomes a child job of its parent and is cancelled when its
 * parent fails or is cancelled. All this supervisor's children are cancelled in this case, too. The invocation of
 * [cancel][Job.cancel] with exception (other than [CancellationException]) on this supervisor job also cancels parent.
 *
 * @param parent an optional parent job.
 */
@Suppress("FunctionName")
public fun SupervisorJob(parent: Job? = null) : CompletableJob = SupervisorJobImpl(parent)
```

​		该函数创建了一个处于 `active` 状态的`supervisor job`。如前所述， `Job` 是有父子关系的，如果子`Job` 失败了父`Job`会自动失败，这种默认的行为可能不是我们期望的。比如在 `Activity` 中有两个子`Job`分别获取一篇文章的评论内容和作者信息。如果其中一个失败了，我们并不希望父`Job`自动取消，这样会导致另外一个子Job也被取消。而`SupervisorJob`就是这么一个特殊的 `Job`，里面的子`Job`不相互影响，一个子`Job`失败了，不影响其他子`Job`的执行。`SupervisorJob(parent:Job?)` 具有一个`parent`参数，如果指定了这个参数，则所返回的 `Job` 就是参数 `parent` 的子`Job`。如果 `Parent Job` 失败了或者取消了，则这个 `Supervisor Job` 也会被取消。当 `Supervisor Job` 被取消后，所有 `Supervisor Job` 的子`Job`也会被取消。

 `MainScope()` 的实现就使用了 `SupervisorJob` 和一个 `Main Dispatcher`：

```kotlin
/**
 * Creates the main [CoroutineScope] for UI components.
 *
 * Example of use:
 * ```
 * class MyAndroidActivity {
 *     private val scope = MainScope()
 *
 *     override fun onDestroy() {
 *         super.onDestroy()
 *         scope.cancel()
 *     }
 * }
 * ```
 *
 * The resulting scope has [SupervisorJob] and [Dispatchers.Main] context elements.
 * If you want to append additional elements to the main scope, use [CoroutineScope.plus] operator:
 * `val scope = MainScope() + CoroutineName("MyActivity")`.
 */
@Suppress("FunctionName")
public fun MainScope(): CoroutineScope = ContextScope(SupervisorJob() + Dispatchers.Main)
```

## CoroutineDispatcher - 调度器

>  `CoroutineDispatcher` 定义了 Coroutine 执行的线程。`CoroutineDispatcher` 可以限定 `Coroutine` 在某一个线程执行、也可以分配到一个线程池来执行、也可以不限制其执行的线程。
>
> `CoroutineDispatcher` 是一个抽象类，所有 `dispatcher` 都应该继承这个类来实现对应的功能。`Dispatchers` 是一个标准库中帮我们封装了切换线程的帮助类，可以简单理解为一个线程池。它的实现如下：

- **Dispatchers.Default**

   默认的调度器，适合处理后台计算，是一个`CPU`密集型任务调度器。如果创建 `Coroutine` 的时候没有指定 `dispatcher`，则一般默认使用这个作为默认值。`Default dispatcher` 使用一个共享的后台线程池来运行里面的任务。注意它和`IO`共享线程池，只不过限制了最大并发数不同。

- **Dispatchers.IO**

   顾名思义这是用来执行阻塞 `IO` 操作的，是和`Default`共用一个共享的线程池来执行里面的任务。根据同时运行的任务数量，在需要的时候会创建额外的线程，当任务执行完毕后会释放不需要的线程。

- **Dispatchers.Unconfined**

  由于`Dispatchers.Unconfined`未定义线程池，所以执行的时候默认在启动线程。遇到第一个挂起点，之后由调用`resume`的线程决定恢复协程的线程。

- **Dispatchers.Main**：

  指定执行的线程是主线程，在`Android`上就是`UI`线程·

由于`子Coroutine` 会继承`父Coroutine` 的 `context`，所以为了方便使用，我们一般会在 `父Coroutine` 上设定一个 `Dispatcher`，然后所有 `子Coroutine` 自动使用这个 `Dispatcher`。

## CoroutineStart - 协程启动模式

- **CoroutineStart.DEFAULT**:

  协程创建后立即开始调度，在调度前如果协程被取消，其将直接进入取消响应的状态

  虽然是立即调度，但也有可能在执行前被取消

- **CoroutineStart.ATOMIC**:

  协程创建后立即开始调度，协程执行到第一个挂起点之前不响应取消

  虽然是立即调度，但其将调度和执行两个步骤合二为一了，就像它的名字一样，其保证调度和执行是原子操作，因此协程也一定会执行

- **CoroutineStart.LAZY**:

  只要协程被需要时，包括主动调用该协程的start、join或者await等函数时才会开始调度，如果调度前就被取消，协程将直接进入异常结束状态

- **CoroutineStart.UNDISPATCHED**:

  协程创建后立即在当前函数调用栈中执行，直到遇到第一个真正挂起的点

  是立即执行，因此协程一定会执行

>这些启动模式的设计主要是为了应对某些特殊的场景。业务开发实践中通常使用**DEFAULT**和**LAZY**这两个启动模式就够了

## CoroutineScope - 协程作用域

> ​		`Coroutine` 是轻量级的线程，并不意味着就不消耗系统资源。 当异步操作比较耗时的时候，或者当异步操作出现错误的时候，需要把这个 `Coroutine` 取消掉来释放系统资源。在 Android 环境中，通常每个界面（`Activity`、Fragment 等）启动的 `Coroutine` 只在该界面有意义，如果用户在等待 `Coroutine` 执行的时候退出了这个界面，则再继续执行这个 `Coroutine` 可能是没必要的。另外 `Coroutine` 也需要在适当的 `context` 中执行，否则会出现错误，比如在非 `UI` 线程去访问 `View`。 所以 Coroutine 在设计的时候，要求在一个范围（`Scope`）内执行，这样当这个 Scope 取消的时候，里面所有的`子 Coroutine` 也自动取消。所以要使用 `Coroutine` 必须要先创建一个对应的 `CoroutineScope`。

官方框架在实现复合协程的过程中也提供了作用域，主要用以明确写成之间的父子关系，以及对于取消或者异常处理等方面的传播行为。该作用域包括以下三种：

* **顶级作用域**

  没有父协程的协程所在的作用域为顶级作用域。

* **协同作用域**

  协程中启动新的协程，新协程为所在协程的子协程，这种情况下，子协程所在的作用域默认为协同作用域。此时子协程抛出的未捕获异常，都将传递给父协程处理，父协程同时也会被取消。

* **主从作用域**

  与协同作用域在协程的父子关系上一致，区别在于，处于该作用域下的协程出现未捕获的异常时，不会将异常向上传递给父协程。

除了三种作用域中提到的行为以外，父子协程之间还存在以下规则：

- 父协程被取消，则所有子协程均被取消。由于协同作用域和主从作用域中都存在父子协程关系，因此此条规则都适用。
- 父协程需要等待子协程执行完毕之后才会最终进入完成状态，不管父协程自身的协程体是否已经执行完。
- 子协程会继承父协程的协程上下文中的元素，如果自身有相同`key`的成员，则覆盖对应的`key`，覆盖的效果仅限自身范围内有效。







# 参考及摘录

[掌握Kotlin Coroutine之 Job&Deferred](http://blog.chengyunfeng.com/?p=1087)

霍丙乾 - 《深入理解Kotlin协程》