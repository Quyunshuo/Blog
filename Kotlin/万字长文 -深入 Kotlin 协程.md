# 前言

> ​		本篇文章是作为我的一个学习记录，写成文章也是为了更好的加深记忆和理解，也是为了分享知识。本文的定位是协程的稍微深入的全面知识，也会示例一些简单的使用，这里不对`suspend`讲解，因为有更好的博文，下文中给出了链接，也不对协程的高级用法做阐述（**热数据通道Channel**、**冷数据流Flow.**..），本文主要讲协程稍微深入的全面知识。

# Kotlin Coroutine 简介

> ​		**Kotlin** 中的协程提供了一种全新处理并发的方式，您可以在 **Android** 平台上使用它来简化异步执行的代码。协程是从 **Kotlin 1.3** 版本开始引入，但这一概念在编程世界诞生的黎明之际就有了，最早使用协程的编程语言可以追溯到 **1967** 年的 **Simula** 语言。
>
> ​		在过去几年间，协程这个概念发展势头迅猛，现已经被诸多主流编程语言采用，比如 **Javascript**、**C#**、**Python**、Ruby 以及 **Go** 等。**Kotlin** 的协程是基于来自其他语言的既定概念。
>
> ​		在 **Android** 平台上，协程主要用来解决两个问题: 
>
> - **处理耗时任务 (Long running tasks)**，这种任务常常会阻塞住主线程；
> - **保证主线程安全 (Main-safety)** ，即确保安全地从主线程调用任何 **suspend** 函数。

## Kotlin Coroutine Version

> **Kotlin Version**: 1.4.32
>
> **Coroutine Version**: 1.4.3

## Kotlin Coroutine 生态

![协程生态.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af760b94ac4f4cab879d55cad89058a6~tplv-k3u1fbpfcp-watermark.image)

kotlin的协程实现分为了两个层次：

* **基础设施层：**

  标准库的协程API，主要对协程提供了概念和语义上最基本的支持

* **业务框架层 kotlin.coroutines：**

  协程的上层框架支持，也是我们日常开发使用的库

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
  
    // lifecycle对于协程的扩展封装
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.2.0"
}
```

# Coroutine 基本使用

## suspend

> ​		关于`suspend`挂起函数，这里不展开去讲，因为有更好的博文，那当然是扔物线凯哥的博文，最初我也是跟随凯哥的视频去学习的写成，大家可以去扔物线的网站去学习下协程的 `suspend` 或其他关于协程的知识，下面放上链接：
>
> [扔物线 - Kotlin 协程的挂起好神奇好难懂？今天我把它的皮给扒了](https://rengwuxian.com/kotlin-xie-cheng-de-gua-qi/)

## 创建协程

> ​		创建协程的方式有很多种，这里不延伸协程的高级用法（**热数据通道Channel**、**冷数据流Flow.**..），也许以后会在文章里补充或者新写文章来专门讲解，创建协程这里介绍常用的两种方式：
>
> - **CoroutineScope.launch()**
> - **CoroutineScope.async()**
>
> 这是常用的协程创建方式，**launch** 构建器适合执行 "一劳永逸" 的工作，意思就是说它可以启动新协程而不将结果返回给调用方；**async** 构建器可启动新协程并允许您使用一个名为 `await` 的挂起函数返回 `result`。 **launch** 和 **async** 之间的很大差异是它们对异常的处理方式不同。**async** 期望最终是通过调用 `await` 来获取结果 (或者异常)，所以默认情况下它不会抛出异常。这意味着如果使用 **async** 启动新的协程，它会静默地将异常丢弃。

### CoroutineScope.launch()

直接上代码：

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import kotlinx.coroutines.*

class MainActivity : AppCompatActivity() {

    /**
     * 使用官方库的 MainScope()获取一个协程作用域用于创建协程
     */
    private val mScope = MainScope()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 创建一个默认参数的协程，其默认的调度模式为Main 也就是说该协程的线程环境是Main线程
        val job1 = mScope.launch {
            // 这里就是协程体

            // 延迟1000毫秒  delay是一个挂起函数
            // 在这1000毫秒内该协程所处的线程不会阻塞
            // 协程将线程的执行权交出去，该线程该干嘛干嘛，到时间后会恢复至此继续向下执行
            delay(1000)
        }

        // 创建一个指定了调度模式的协程，该协程的运行线程为IO线程
        val job2 = mScope.launch(Dispatchers.IO) {

            // 此处是IO线程模式

            // 切线程 将协程所处的线程环境切至指定的调度模式Main
            withContext(Dispatchers.Main) {
                // 现在这里就是Main线程了  可以在此进行UI操作了
            }
        }

        // 下面直接看一个例子： 从网络中获取数据  并更新UI
        // 该例子不会阻塞主线程
        mScope.launch(Dispatchers.IO) {
            // 执行getUserInfo方法时会将线程切至IO去执行
            val userInfo = getUserInfo()
            // 获取完数据后 切至Main线程进行更新UI
            withContext(Dispatchers.Main) {
                // 更新UI
            }
        }
    }

    /**
     * 获取用户信息 该函数模拟IO获取数据
     * @return String
     */
    private suspend fun getUserInfo(): String {
        return withContext(Dispatchers.IO) {
            delay(2000)
            "Kotlin"
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        // 取消协程 防止协程泄漏  如果使用lifecycleScope则不需要手动取消
        mScope.cancel()
    }
}
```

上面的代码中，给出了一些代码示例，其实协程的简单使用非常简单，你甚至完全不需要担心其他的东西，你只需要记得及时取消协程就ok，如果你使用`lifecycleScope`或者`viewModelScope`你连取消都不用自己管，界面或`ViewModel`被销毁时，会自动帮你把协程取消掉。使用协程只需要会创建、会切线程、懂四种调度模式，基本就ok了，基本开发已满足。

### CoroutineScope.async()

`async`主要用于获取返回值和并发，直接上代码：

```kotlin
fun asyncTest() {
    mScope.launch {
        // 开启一个IO模式的线程 并返回一个Deferred，Deferred可以用来获取返回值
        // 代码执行到此处时会新开一个协程 然后去执行协程体  父协程的代码会接着往下走
        val deferred = async(Dispatchers.IO) {
            // 模拟耗时
            delay(2000)
            // 返回一个值
            "Quyunshuo"
        }
        // 等待async执行完成获取返回值 此处并不会阻塞线程  而是挂起 将线程的执行权交出去
        // 等到async的协程体执行完毕后  会恢复协程继续往下执行
        val date = deferred.await()
    }
}
```

上面的代码主要展示`async`的返回值功能，需要与`await()`挂起函数结合使用

下面展示`async`的并发能力：

```kotlin
fun asyncTest2() {
    mScope.launch {
        // 此处有一个需求  同时请求5个接口  并且将返回值拼接起来

        val job1 = async {
            // 请求1 
            delay(5000)
            "1"
        }
        val job2 = async {
            // 请求2 
            delay(5000)
            "2"
        }
        val job3 = async {
            // 请求3 
            delay(5000)
            "3"
        }
        val job4 = async {
            // 请求4 
            delay(5000)
            "4"
        }
        val job5 = async {
            // 请求5
            delay(5000)
            "5"
        }

        // 代码执行到此处时  5个请求已经同时在执行了
        // 等待各job执行完 将结果合并
        Log.d(
            "TAG",
            "asyncTest2: ${job1.await()} ${job2.await()} ${job3.await()} ${job4.await()} ${job5.await()}"
        )

        // 因为我们设置的模拟时间都是5000毫秒  所以当job1执行完时  其他job也均执行完成
    }
}
```

上面的代码就是一个简单的并发示例，是不是感觉十分的简单，协程的优势立马凸显出来了。

这就是最基本的协程使用，关于作用域，更推荐的是在UI组件中使用`LifecycleOwner.lifecycleScope`，在`ViewModel`中使用`ViewModel.viewModelScope`

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

> ​		`CoroutineContext`即协程的上下文，是 Kotlin 协程的一个基本结构单元。巧妙的运用协程上下文是至关重要的，以此来实现正确的线程行为、生命周期、异常以及调试。它包含用户定义的一些数据集合，这些数据与协程密切相关。它是一个有索引的 `Element` 实例集合。这个有索引的集合类似于一个介于 `set` 和 map之间的数据结构。每个 `element` 在这个集合有一个唯一的 Key 。当多个 `element` 的 key 的引用相同，则代表属于集合里同一个 `element`。它由如下几项构成:
>
> - **Job**: 控制协程的生命周期；
> - **CoroutineDispatcher**: 向合适的线程分发任务；
> - **CoroutineName**: 协程的名称，调试的时候很有用；
> - **CoroutineExceptionHandler**: 处理未被捕捉的异常。
>
> `CoroutineContext` 有两个非常重要的元素 — `Job` 和 `Dispatcher`，`Job` 是当前的 `Coroutine` 实例而 `Dispatcher` 决定了当前 `Coroutine` 执行的线程，还可以添加`CoroutineName`，用于调试，添加 `CoroutineExceptionHandler` 用于捕获异常，它们都实现了`Element`接口。看一个例子：

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

> 在任务层级中，每个协程都会有一个父级对象，要么是 `CoroutineScope` 或者另外一个 `coroutine`。然而，实际上协程的父级 `CoroutineContext` 和父级协程的 `CoroutineContext` 是不一样的，因为有如下的公式:

**父级上下文 = 默认值 + 继承的 CoroutineContext + 参数**

其中:

- **一些元素包含默认值: Dispatchers.Default 是默认的 CoroutineDispatcher，以及 "coroutine" 作为默认的 CoroutineName；**
- **继承的 CoroutineContext 是 CoroutineScope 或者其父协程的 CoroutineContext；**
- **传入协程 builder 的参数的优先级高于继承的上下文参数，因此会覆盖对应的参数值。**

**请注意:** `CoroutineContext` 可以使用 " + " 运算符进行合并。由于 `CoroutineContext` 是由一组元素组成的，所以加号右侧的元素会覆盖加号左侧的元素，进而组成新创建的 `CoroutineContext`。比如，`(Dispatchers.Main, "name") + (Dispatchers.IO) = (Dispatchers.IO, "name")。`

## Job & Deferred - 任务

> ​		`Job` 用于处理协程。对于每一个所创建的协程 (通过 `launch` 或者 async)，它会返回一个 `Job`实例，该实例是协程的唯一标识，并且负责管理协程的生命周期
>
> ​		`CoroutineScope.launch` 函数返回的是一个 `Job` 对象，代表一个异步的任务。`Job` 具有生命周期并且可以取消。 `Job` 还可以有层级关系，一个`Job`可以包含多个子`Job`，当父`Job`被取消后，所有的子`Job`也会被自动取消；当子`Job`被取消或者出现异常后父`Job`也会被取消。
>
> ​		除了通过 `CoroutineScope.launch` 来创建`Job`对象之外，还可以通过 `Job()` 工厂方法来创建该对象。默认情况下，子`Job`的失败将会导致父`Job`被取消，这种默认的行为可以通过 `SupervisorJob` 来修改。
>
> ​		具有多个子 `Job` 的父`Job` 会等待所有子`Job`完成(或者取消)后，自己才会执行完成

### Job 的状态

> ​		一个任务可以包含一系列状态: 新创建 (**New**)、活跃 (**Active**)、完成中 (**Completing**)、已完成 (Completed)、取消中 (**Cancelling**) 和已取消 (**Cancelled**)。虽然我们无法直接访问这些状态，但是我们可以访问 `Job` 的属性: `isActive`、`isCancelled` 和 `isCompleted`。
>
> ​		如果协程处于活跃状态，协程运行出错或者调用 `job.cancel()` 都会将当前任务置为取消中 (**Cancelling**) 状态 (`isActive = false, isCancelled = true`)。当所有的子协程都完成后，协程会进入已取消 (**Cancelled**) 状态，此时 `isCompleted = true`。

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

但是`SupervisorJob`是很容易被误解的，它和协程异常处理、子协程所属`Job`类型还有域有很多让人混淆的地方，具体异常处理可以看Google的这一篇文章：[协程中的取消和异常 | 异常处理详解](https://blog.csdn.net/jILRvRTrc/article/details/107437548)

## CoroutineDispatcher - 调度器

>  `CoroutineDispatcher` 定义了 Coroutine 执行的线程。`CoroutineDispatcher` 可以限定 `Coroutine` 在某一个线程执行、也可以分配到一个线程池来执行、也可以不限制其执行的线程。
>
> `CoroutineDispatcher` 是一个抽象类，所有 `dispatcher` 都应该继承这个类来实现对应的功能。`Dispatchers` 是一个标准库中帮我们封装了切换线程的帮助类，可以简单理解为一个线程池。它的实现如下：

![dispatchers.webp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a31f4fca427a4c67b9bbda39ee11887c~tplv-k3u1fbpfcp-watermark.image)

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

> ​		定义协程必须指定其 `CoroutineScope` 。`CoroutineScope` 可以对协程进行追踪，即使协程被挂起也是如此。同调度程序 (`Dispatcher`) 不同，`CoroutineScope` 并不运行协程，它只是确保您不会失去对协程的追踪。为了确保所有的协程都会被追踪，`Kotlin` 不允许在没有使用 `CoroutineScope` 的情况下启动新的协程。`CoroutineScope` 可被看作是一个具有超能力的 `ExecutorService` 的轻量级版本。`CoroutineScope` 会跟踪所有协程，同样它还可以取消由它所启动的所有协程。这在 `Android` 开发中非常有用，比如它能够在用户离开界面时停止执行协程。
>
> ​		`Coroutine` 是轻量级的线程，并不意味着就不消耗系统资源。 当异步操作比较耗时的时候，或者当异步操作出现错误的时候，需要把这个 `Coroutine` 取消掉来释放系统资源。在 `Android` 环境中，通常每个界面（`Activity`、`Fragment` 等）启动的 `Coroutine` 只在该界面有意义，如果用户在等待 `Coroutine` 执行的时候退出了这个界面，则再继续执行这个 `Coroutine` 可能是没必要的。另外 `Coroutine` 也需要在适当的 `context` 中执行，否则会出现错误，比如在非 `UI` 线程去访问 `View`。 所以 `Coroutine` 在设计的时候，要求在一个范围（`Scope`）内执行，这样当这个 `Scope` 取消的时候，里面所有的`子 Coroutine` 也自动取消。所以要使用 `Coroutine` 必须要先创建一个对应的 `CoroutineScope`。

### **CoroutineScope 接口**

```kotlin
public interface CoroutineScope {
    public val coroutineContext: CoroutineContext
}
```

`CoroutineScope` 只是定义了一个新 `Coroutine` 的执行 `Scope`。每个 `coroutine builder` 都是 `CoroutineScope` 的扩展函数，并且自动的继承了当前 `Scope` 的 `coroutineContext` 。

### 分类及行为规则

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

### 常用作用域

> ​		官方库给我们提供了一些作用域可以直接来使用，并且 Android 的Lifecycle Ktx库也封装了更好用的作用域，下面看一下各种作用域

#### GlobalScope - 不推荐使用

```kotlin
public object GlobalScope : CoroutineScope {
    /**
     * Returns [EmptyCoroutineContext].
     */
    override val coroutineContext: CoroutineContext
        get() = EmptyCoroutineContext
}
```

GlobalScope是一个单例实现，源码十分简单，上下文是`EmptyCoroutineContext`，是一个空的上下文，切不包含任何Job，该作用域常被拿来做示例代码，由于 GlobalScope 对象没有和应用生命周期组件相关联，需要自己管理 GlobalScope 所创建的 Coroutine，且`GlobalScope`的生命周期是 process 级别的，所以一般而言我们不推荐使用 GlobalScope 来创建 Coroutine。

#### runBlocking{} - 主要用于测试

```kotlin
/**
 * Runs a new coroutine and **blocks** the current thread _interruptibly_ until its completion.
 * This function should not be used from a coroutine. It is designed to bridge regular blocking code
 * to libraries that are written in suspending style, to be used in `main` functions and in tests.
 *
 * The default [CoroutineDispatcher] for this builder is an internal implementation of event loop that processes continuations
 * in this blocked thread until the completion of this coroutine.
 * See [CoroutineDispatcher] for the other implementations that are provided by `kotlinx.coroutines`.
 *
 * When [CoroutineDispatcher] is explicitly specified in the [context], then the new coroutine runs in the context of
 * the specified dispatcher while the current thread is blocked. If the specified dispatcher is an event loop of another `runBlocking`,
 * then this invocation uses the outer event loop.
 *
 * If this blocked thread is interrupted (see [Thread.interrupt]), then the coroutine job is cancelled and
 * this `runBlocking` invocation throws [InterruptedException].
 *
 * See [newCoroutineContext][CoroutineScope.newCoroutineContext] for a description of debugging facilities that are available
 * for a newly created coroutine.
 *
 * @param context the context of the coroutine. The default value is an event loop on the current thread.
 * @param block the coroutine code.
 */
@Throws(InterruptedException::class)
public fun <T> runBlocking(context: CoroutineContext = EmptyCoroutineContext, block: suspend CoroutineScope.() -> T): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    val currentThread = Thread.currentThread()
    val contextInterceptor = context[ContinuationInterceptor]
    val eventLoop: EventLoop?
    val newContext: CoroutineContext
    if (contextInterceptor == null) {
        // create or use private event loop if no dispatcher is specified
        eventLoop = ThreadLocalEventLoop.eventLoop
        newContext = GlobalScope.newCoroutineContext(context + eventLoop)
    } else {
        // See if context's interceptor is an event loop that we shall use (to support TestContext)
        // or take an existing thread-local event loop if present to avoid blocking it (but don't create one)
        eventLoop = (contextInterceptor as? EventLoop)?.takeIf { it.shouldBeProcessedFromContext() }
            ?: ThreadLocalEventLoop.currentOrNull()
        newContext = GlobalScope.newCoroutineContext(context)
    }
    val coroutine = BlockingCoroutine<T>(newContext, currentThread, eventLoop)
    coroutine.start(CoroutineStart.DEFAULT, coroutine, block)
    return coroutine.joinBlocking()
}
```

这是一个顶层函数，从源码的注释中我们可以得到一些信息，运行一个新的协程并且阻塞当前可中断的线程直至协程执行完成，该函数不应从一个协程中使用，该函数被设计用于桥接普通阻塞代码到以挂起风格（`suspending style`）编写的库，以用于主函数与测试。该函数主要用于测试，不适用于日常开发，该协程会阻塞当前线程直到协程体执行完成。

#### MainScope() - 可用于开发

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

该函数是一个顶层函数，用于返回一个上下文是`SupervisorJob() + Dispatchers.Main`的作用域，该作用域常被使用在Activity/Fragment，并且在界面销毁时要调用`fun CoroutineScope.cancel(cause: CancellationException? = null)`对协程进行取消，这是官方库中可以在开发中使用的一个用于获取作用域的顶层函数，使用示例在官方库的代码注释中已经给出，上面的源码中也有，使用起来也是十分的方便。

#### LifecycleOwner.lifecycleScope - 推荐使用

```kotlin
/**
 * [CoroutineScope] tied to this [LifecycleOwner]'s [Lifecycle].
 *
 * This scope will be cancelled when the [Lifecycle] is destroyed.
 *
 * This scope is bound to
 * [Dispatchers.Main.immediate][kotlinx.coroutines.MainCoroutineDispatcher.immediate].
 */
val LifecycleOwner.lifecycleScope: LifecycleCoroutineScope
    get() = lifecycle.coroutineScope
```

该扩展属性是 `Android` 的`Lifecycle Ktx`库提供的具有生命周期感知的协程作用域，它与`LifecycleOwner`的`Lifecycle`绑定，Lifecycle被销毁时，此作用域将被取消。这是在`Activity/Fragment`中推荐使用的作用域，因为它会与当前的UI组件绑定生命周期，界面销毁时该协程作用域将被取消，不会造成协程泄漏，相同作用的还有下文提到的`ViewModel.viewModelScope`。

#### ViewModel.viewModelScope - 推荐使用

```kotlin
/**
 * [CoroutineScope] tied to this [ViewModel].
 * This scope will be canceled when ViewModel will be cleared, i.e [ViewModel.onCleared] is called
 *
 * This scope is bound to
 * [Dispatchers.Main.immediate][kotlinx.coroutines.MainCoroutineDispatcher.immediate]
 */
val ViewModel.viewModelScope: CoroutineScope
        get() {
            val scope: CoroutineScope? = this.getTag(JOB_KEY)
            if (scope != null) {
                return scope
            }
            return setTagIfAbsent(JOB_KEY,
                CloseableCoroutineScope(SupervisorJob() + Dispatchers.Main.immediate))
        }
```

该扩展属性和上文中提到的`LifecycleOwner.lifecycleScope`基本一致，它是`ViewModel`的扩展属性，也是来自`Android` 的`Lifecycle Ktx`库，它能够在此`ViewModel`销毁时自动取消，同样不会造成协程泄漏。该扩展属性返回的作用域的上下文同样是`SupervisorJob() + Dispatchers.Main.immediate`

####  coroutineScope & supervisorScope

```kotlin
/**
 * Creates a [CoroutineScope] with [SupervisorJob] and calls the specified suspend block with this scope.
 * The provided scope inherits its [coroutineContext][CoroutineScope.coroutineContext] from the outer scope, but overrides
 * context's [Job] with [SupervisorJob].
 *
 * A failure of a child does not cause this scope to fail and does not affect its other children,
 * so a custom policy for handling failures of its children can be implemented. See [SupervisorJob] for details.
 * A failure of the scope itself (exception thrown in the [block] or cancellation) fails the scope with all its children,
 * but does not cancel parent job.
 */
public suspend fun <R> supervisorScope(block: suspend CoroutineScope.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return suspendCoroutineUninterceptedOrReturn { uCont ->
        val coroutine = SupervisorCoroutine(uCont.context, uCont)
        coroutine.startUndispatchedOrReturn(coroutine, block)
    }
}



/**
 * Creates a [CoroutineScope] and calls the specified suspend block with this scope.
 * The provided scope inherits its [coroutineContext][CoroutineScope.coroutineContext] from the outer scope, but overrides
 * the context's [Job].
 *
 * This function is designed for _parallel decomposition_ of work. When any child coroutine in this scope fails,
 * this scope fails and all the rest of the children are cancelled (for a different behavior see [supervisorScope]).
 * This function returns as soon as the given block and all its children coroutines are completed.
 * A usage example of a scope looks like this:
 *
 * ```
 * suspend fun showSomeData() = coroutineScope {
 *     val data = async(Dispatchers.IO) { // <- extension on current scope
 *      ... load some UI data for the Main thread ...
 *     }
 *
 *     withContext(Dispatchers.Main) {
 *         doSomeWork()
 *         val result = data.await()
 *         display(result)
 *     }
 * }
 * ```
 *
 * The scope in this example has the following semantics:
 * 1) `showSomeData` returns as soon as the data is loaded and displayed in the UI.
 * 2) If `doSomeWork` throws an exception, then the `async` task is cancelled and `showSomeData` rethrows that exception.
 * 3) If the outer scope of `showSomeData` is cancelled, both started `async` and `withContext` blocks are cancelled.
 * 4) If the `async` block fails, `withContext` will be cancelled.
 *
 * The method may throw a [CancellationException] if the current job was cancelled externally
 * or may throw a corresponding unhandled [Throwable] if there is any unhandled exception in this scope
 * (for example, from a crashed coroutine that was started with [launch][CoroutineScope.launch] in this scope).
 */
public suspend fun <R> coroutineScope(block: suspend CoroutineScope.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return suspendCoroutineUninterceptedOrReturn { uCont ->
        val coroutine = ScopeCoroutine(uCont.context, uCont)
        coroutine.startUndispatchedOrReturn(coroutine, block)
    }
}
```

首先这两个函数都是挂起函数，需要运行在协程内或挂起函数内。`supervisorScope`属于主从作用域，会继承父协程的上下文，它的特点就是子协程的异常不会影响父协程，它的设计应用场景多用于子协程为独立对等的任务实体的时候，比如一个下载器，每一个子协程都是一个下载任务，当一个下载任务异常时，它不应该影响其他的下载任务。`coroutineScope`和`supervisorScope`都会返回一个作用域，它俩的差别就是异常传播：`coroutineScope` 内部的异常会向上传播，子协程未捕获的异常会向上传递给父协程，任何一个子协程异常退出，会导致整体的退出；`supervisorScope` 内部的异常不会向上传播，一个子协程异常退出，不会影响父协程和兄弟协程的运行。

## 协程的取消和异常

> ​		普通协程如果产生未处理异常会将此异常传播至它的父协程，然后父协程会取消所有的子协程、取消自己、将异常继续向上传递。下面拿一个官方的图来示例这个过程：

![coroutine_e.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28647715daef471ebdeef6883d823716~tplv-k3u1fbpfcp-watermark.image)

这种情况有的时候并不是我们想要的，我们更希望一个协程在产生异常时，不影响其他协程的执行，在上文中我们也提到了一些解决方案，下面我们就在实践一下。

**使用SupervisorJob****

在上文中我们也对这个顶层函数做了讲解，那如何使用呢？直接上代码：

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.util.Log
import kotlinx.coroutines.*

class MainActivity : AppCompatActivity() {

    /**
     * 使用官方库的 MainScope()获取一个协程作用域用于创建协程
     */
    private val mScope = MainScope()

    companion object {
        const val TAG = "Kotlin Coroutine"
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        mScope.launch(Dispatchers.Default) {
            delay(500)
            Log.e(TAG, "Child 1")
        }

        mScope.launch(Dispatchers.Default) {
            delay(1000)
            Log.e(TAG, "Child 2")
            throw RuntimeException("--> RuntimeException <--")
        }

        mScope.launch(Dispatchers.Default) {
            delay(1500)
            Log.e(TAG, "Child 3")
        }
    }
}


打印结果：
E/Kotlin Coroutine: Child 1
E/Kotlin Coroutine: Child 2
E/AndroidRuntime: FATAL EXCEPTION: DefaultDispatcher-worker-3
    Process: com.quyunshuo.kotlincoroutine, PID: 24240
    java.lang.RuntimeException: --> RuntimeException <--
        at com.quyunshuo.kotlincoroutine.MainActivity$onCreate$2.invokeSuspend(MainActivity.kt:31)
        at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
        at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:106)
        at kotlinx.coroutines.scheduling.CoroutineScheduler.runSafely(CoroutineScheduler.kt:571)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.executeTask(CoroutineScheduler.kt:750)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.runWorker(CoroutineScheduler.kt:678)
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.run(CoroutineScheduler.kt:665)
E/Kotlin Coroutine: Child 3
```

`MainScope()`我们之前提到过了，它的实现就是用了`SupervisorJob`。执行结果就是***Child 2***抛出异常后，***Child 3***正常执行了，但是程序崩了，因为我们没有处理这个异常，下面完善一下代码

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    mScope.launch(Dispatchers.Default) {
        delay(500)
        Log.e(TAG, "Child 1")
    }

  	// 在Child 2的上下文添加了异常处理
    mScope.launch(Dispatchers.Default + CoroutineExceptionHandler { coroutineContext, throwable ->
        Log.e(TAG, "CoroutineExceptionHandler: $throwable")
    }) {
        delay(1000)
        Log.e(TAG, "Child 2")
        throw RuntimeException("--> RuntimeException <--")
    }

    mScope.launch(Dispatchers.Default) {
        delay(1500)
        Log.e(TAG, "Child 3")
    }
}


输出结果：
E/Kotlin Coroutine: Child 1
E/Kotlin Coroutine: Child 2
E/Kotlin Coroutine: CoroutineExceptionHandler: java.lang.RuntimeException: --> RuntimeException <--
E/Kotlin Coroutine: Child 3
```

这一次，程序没有崩溃，并且异常处理的打印也输出了，这就达到了我们想要的效果。但是要注意一个事情，这几个子协程的父级是`SupervisorJob`，但是他们再有子协程的话，他们的子协程的父级就不是SupervisorJob了，所以当它们产生异常时，就不是我们演示的效果了。我们使用一个官方的图来解释这个关系：

![child_job.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2364205c632d4282b50c8f2cce6e8df1~tplv-k3u1fbpfcp-watermark.image)

这个图可以说是非常直观了，还是官方🐂。新的协程被创建时，会生成新的 `Job` 实例替代 `SupervisorJob`。

**使用supervisorScope**

这个作用域我们上文中也有提到，使用`supervisorScope`也可以达到我们想要的效果，上代码：

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.util.Log
import kotlinx.coroutines.*

class MainActivity : AppCompatActivity() {

    companion object {
        const val TAG = "Kotlin Coroutine"
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val scope = CoroutineScope(Job() + Dispatchers.Default)

        scope.launch(CoroutineExceptionHandler { coroutineContext, throwable ->
            Log.e(TAG, "CoroutineExceptionHandler: $throwable")
        }) {
            supervisorScope {
                launch {
                    delay(500)
                    Log.e(TAG, "Child 1 ")
                }
                launch {
                    delay(1000)
                    Log.e(TAG, "Child 2 ")
                    throw  RuntimeException("--> RuntimeException <--")
                }
                launch {
                    delay(1500)
                    Log.e(TAG, "Child 3 ")
                }
            }
        }
    }
}

输出结果：
E/Kotlin Coroutine: Child 1 
E/Kotlin Coroutine: Child 2 
E/Kotlin Coroutine: CoroutineExceptionHandler: java.lang.RuntimeException: --> RuntimeException <--
E/Kotlin Coroutine: Child 3 
```

可以看到已经达到了我们想要的效果，但是如果将`supervisorScope`换成`coroutineScope`，结果就不是这样了。最终还是拿官方的图来展示：

![ok.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc9d14675c674fbc92da15e8590c50f8~tplv-k3u1fbpfcp-watermark.image)

# 结语

> 至此文章就已经结束，本文主要是我学习协程的一些记录，分享出来供大家翻阅一下，大家好才是真的好。里面有很多的描述都是摘录自别的文章的，下面也给出了链接，其实官方的文章已经将协程的使用讲的非常全面了，大家可以翻阅一下官方的文章进行学习，虽然可能描述的不是很详细，但是该有的细节都提到了。
>
> ​		后续有时间可能会出一些协程的高级用法的文章，比如协程的冷数据流Flow，这个在我们的项目里也已经用上了，没错，是我引入的😁。总体来说，协程简单使用非常简单，但是想用好，还是需要下一定的功夫去研究的，但是还是逃不过真香定律，大家赶紧学习用起来吧。

**我的其他文章**：

- [一个 Android MVVM 组件化架构框架](https://juejin.cn/post/6866628586414997512)
- [解锁管理EventBus注册新姿势——自定义注解+反射](https://juejin.cn/post/6945058494550638629)
- [万字长文 - 史上最全ConstraintLayout（约束布局）使用详解](https://juejin.cn/post/6949186887609221133)

# 参考及摘录

[掌握Kotlin Coroutine之 Job&Deferred](http://blog.chengyunfeng.com/?p=1087)

霍丙乾 - 《深入理解Kotlin协程》

[谷歌开发者 - 在 Android 开发中使用协程 | 背景介绍](https://mp.weixin.qq.com/s?__biz=MzAwODY4OTk2Mg==&mid=2652052998&idx=2&sn=18715a7e33b7f7a5878bd301e9f8f935&scene=21#wechat_redirect)

[谷歌开发者 - 协程中的取消和异常 | 核心概念介绍](https://mp.weixin.qq.com/s?__biz=MzAwODY4OTk2Mg%3D%3D&chksm=808c8358b7fb0a4ee7ab15a9655543c501b3e1fd7c6c5f84f9e151a58c93264ff74066246696&idx=2&mid=2652054301&scene=21&sn=917dddffbdb4d97f950f70dfc570c021#wechat_redirect)

