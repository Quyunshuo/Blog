## Toast 介绍
> 消息框可以在一个小型弹出式窗口中提供与操作有关的简单反馈。它只会填充消息所需的空间大小，并且当前 Activity 会一直显示及供用户与之互动。超时后，消息框会自动消失。样式如下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08d2531d4b8f4265956b91c30c9f5e08~tplv-k3u1fbpfcp-watermark.image)

## Toast 替代方案 Snackbar

在 **Android Developers** 中，官方推荐使用 `Snackbar` 来替代 `Toast`，并且在 **Android 11** 中已经进一步阉割限制 `Toast` 的使用，原文如下：

> 如果您的应用在前台运行，请考虑使用信息提示控件替代消息框。信息提示控件中包含用户可操作的选项，可提供更好的应用体验。
> 如果您的应用在后台运行，并且您希望用户执行某项操作，请改用通知。

官方在新的 **Android** 版本中已经有意的将 `Toast` 的功能进行限制，并且推荐我们使用 `Snackbar` 来完成在应用前台时的消息反馈，说明在将来会对 `Toast` 进一步限制，我们自定义的需求将更不好实现。所以我们开发人员以及产品设计等可以参照官方给的建议进行重新定义这种简单的反馈。

**Snackbar 样式如下图：**

![unnamed.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec2aaef394db4cbb8ec6eabf89372bf9~tplv-k3u1fbpfcp-watermark.image)

## Toast 的使用方式

### 创建

`Toast` 的创建十分的简单，甚至在 **Android Studio** 中已经提供了快捷方式进行创建，只需要在编辑器中打出 `toast` 就会出现代码提示，选择后就会自动出现已经写好的创建代码，只需要传入相应参数即可：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92e63659a62a48bda72ffc65465bf4f1~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ac24bc1378347b280cdcd8b01e7a4be~tplv-k3u1fbpfcp-watermark.image)

从官方提供的代码中我们可以看到，创建 `Toast` 使用的是 `makeText()` 静态方法：

``` java
/**
 * Make a standard toast that just contains text from a resource.
 *
 * @param context  The context to use.  Usually your {@link android.app.Application}
 *                 or {@link android.app.Activity} object.
 * @param resId    The resource id of the string resource to use.  Can be formatted text.
 * @param duration How long to display the message.  Either {@link #LENGTH_SHORT} or
 *                 {@link #LENGTH_LONG}
 *
 * @throws Resources.NotFoundException if the resource can't be found.
 */
public static Toast makeText(Context context, @StringRes int resId, @Duration int duration)
                            throws Resources.NotFoundException {
    return makeText(context, context.getResources().getText(resId), duration);
}

/**
 * Make a standard toast that just contains text from a resource.
 *
 * @param context  The context to use.  Usually your {@link android.app.Application}
 *                 or {@link android.app.Activity} object.
 * @param resId    The resource id of the string resource to use.  Can be formatted text.
 * @param duration How long to display the message.  Either {@link #LENGTH_SHORT} or
 *                 {@link #LENGTH_LONG}
 *
 * @throws Resources.NotFoundException if the resource can't be found.
 */
public static Toast makeText(Context context, @StringRes int resId, @Duration int duration)
                            throws Resources.NotFoundException {
    return makeText(context, context.getResources().getText(resId), duration);
}
```

该方法接收三个参数：`Context` **上下文**、**提示的文字**、**时间**，其中时间有两个枚举值：`LENGTH_SHORT`、`LENGTH_LONG`，分别是短时间和长时间。

### 改变提示文案

如果我们需要弹一个文案不一样的 `Toast` 其实大可不必新建一个，直接复用之前的 `Toast` 对象即可，通过 `setText()` 方法进行设置新的提示文案。

### 改变位置

改变位置可以使用 `setGravity(int gravity, int xOffset, int yOffset)`方法，但是该方法在普通`Toast` 上 当 **targetSdkVersion** 为 **R** 或更高时，将失去效果，下文会详细说明。

### 自定义 Toast

当默认的样式满足不了我们的时候就需要进行自定义 `Toast`，自定义 `Toast` 样式使用如下方法：

``` java
/**
 * Set the view to show.
 *
 * @see #getView
 * @deprecated Custom toast views are deprecated. Apps can create a standard text toast with the
 *      {@link #makeText(Context, CharSequence, int)} method, or use a
 *      <a href="{@docRoot}reference/com/google/android/material/snackbar/Snackbar">Snackbar</a>
 *      when in the foreground. Starting from Android {@link Build.VERSION_CODES#R}, apps
 *      targeting API level {@link Build.VERSION_CODES#R} or higher that are in the background
 *      will not have custom toast views displayed.
 */
@Deprecated
public void setView(View view) {
    mNextView = view;
}
```

我把源码全部粘出来就是为了让大家看一下官方的注释，在注释中我们可以看到，官方推荐使用 `makeText(Context, CharSequence, int)` 来创建标准的 `Toast`，并且该 `setView(View view)` 方法已经标记为废弃。

所以这里我也不详细介绍自定义的细节了，一是大同小异，二是这种方式已经标记废弃了。

### 线程问题

总所周知视图的更新必须要在**UI**线程操作，`Toast` 也不例外，所以在使用的时候要么使用别人封装好的支持子线程弹出的工具类，要么就要注意不要在子线程直接弹。

## 内存泄漏问题

之前在写业务的时候使用 `Toast` 引发了内存泄漏，当时非常不解，具体原因是因为如果在创建 `Toast` 的时候传入的 `Context` 是 `Activity` 的时候就很容易引发内存泄漏。非常常见的一个场景就是在页面即将销毁前弹出了 `Toast`，页面走了 `onDestroy()` ,但是此时 `Toast` 并没有消失，还持有 `Activity` 的引用，这不直接GG？？所以我们在使用 `Toast` 的时候一定不要传入 `Activity` 的 `Context`，替换为 `Application` 的 `Context` 就可以解决这个问题。关于内存泄漏的细节可以看这篇文章的源码解析：[Toast 为什么会造成内存泄漏](https://blog.csdn.net/Deaht_Huimie/article/details/104434510/)。

## Android 11 Toast 的API变更

### setView()弃用

如上文中所说的，在 **Android 11** 开始，已经废弃了 `setView()` 方法，不久的将来该方法将彻底消失在源码中。

### text toast 不允许自定义

默认的 `Toast` 是 `text toast`，如果想使用自定义的 `toast`，需要调用 `setView()` 方法,在 **targetSdkVersion** 为 **R** 或更高时，调用 `setGravity()` 和 `setMargin()` 方法将不进行任何操作。

### 新增 Toast.Callback 回调

Toast.Callback 用于在Toast 显示和消失时进行回调，使用方式如下：

``` kotlin
toast.addCallback(
    @RequiresApi(Build.VERSION_CODES.R)
    object : Toast.Callback() {
        override fun onToastShown() {
            super.onToastShown()
            Log.d("qqq", "onToastShown: ")
        }

        override fun onToastHidden() {
            super.onToastHidden()
            Log.d("qqq", "onToastShown: ")
        }
    })
```
需要注意的是该回调需要API等级为 Android 11

### 后台不允许弹出自定义 Toast

在 `setView()`方法的注释中我们可以看到这句话：`Starting from Android {@link Build.VERSION_CODES#R}, apps targeting API level {@link Build.VERSION_CODES#R} or higher that are in the background will not have custom toast views displayed.`，也就是说自定义的 `Toast` 将不会在后台显示，但是对于普通 `Toast` 来说不会影响。

## 参考
[Android 11 下 Toast 变化，不能自定义 Toast 了？](https://juejin.cn/post/6844904144424140808#heading-9)

[Android Developers - 消息框概览](https://developer.android.com/guide/topics/ui/notifiers/toasts?hl=zh-cn)