# 国外大厂面试题， 7 个 Android Lifecycle 重要的知识点

![](https://img.hi-dhl.com/202211131955512.png)

> hi 大家好，我是 DHL。公众号：ByteCode ，分享有用、专注有用、有趣的硬核原创内容，Kotlin、Jetpack、性能优化、系统源码、算法及数据结构、大厂面经。
> 原文地址：[国外大厂面试题， 7 个 Android Lifecycle 重要的知识点](https://mp.weixin.qq.com/s/oN4CTWCnEWSc2yf2BG6dew)



习惯性的每天都会打开 medium 看一下技术相关的内容，偶然看到一位大佬分享和 `Android Lifecycle` 相关的面试题，觉得非常的有价值。

在 Android 开发中 `Android Lifecycle` 是非常重要的知识点。但是不幸的是，我发现很多新的 Android 开发对 `Android Lifecycle` 不是很了解，导致在开发中遇到很多奇怪的问题。

分享这些面试题，不仅仅是为了通过面试，更是为了让 Android 开发者基础更加的扎实，防止在开发中遇到很多奇怪的问题。

### 面试题一：Launch Fragment by Default

**问题:**

花几秒钟思考一下，下面的代码有什么问题。

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        supportFragmentManager
           .beginTransaction()
           .replace(R.id.container, MainFragment())
           .commit()
   }
}
```

**错误的回答**

1. 没有使用 KTX 添加 `Fragment`
2. 应该使用 `.add` 而不是 `.replace`

**正确的回答**

如果 Activity 因意外被杀死并被恢复，会再次执行 `onCreate()` 方法，创建新的 Fragment，因此在栈中会存在 2 个  Fragment。在 Fragment 上的任何操作都可能被执行两次，这将会导致出现奇怪的问题。

![](https://img.hi-dhl.com/202211131955513.jpeg)


为了防止 Activity 因意外被杀死而恢复，导致添加新的 Fragment，所以我们可以使用 `stateInstanceState == null` 作为判断条件，防止添加新的 Fragment，因此我们可以将上面的代码简单修改一下。


```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        if (savedInstanceState == null) {
            supportFragmentManager
                .beginTransaction()
                .replace(R.id.container, MainFragment())
                .commit()
        }
   }
}
```

### 面试题二：Create Fragment with Constructor

**问题：**

如果往 Fragment 构造函数中添加参数，花几秒钟思考一下，下面的代码会有什么问题？


```kotlin
supportFragmentManager
    .beginTransaction()
    .replace(R.id.container, MainFragment())
    .commit()
    
class MainFragment(private val repository: Repository): Fragment() {
    
}    
```

**错误的回答**


1. 我们可以使用 `.replace(R.id.container, MainFragment(repository))` 方法来代替
2. 它不会编译和运行

**正确的回答**

我们不应该直接用带参数的构造函数实例化任何 `Fragment()`，如果想使用带参数的构造函数实例化 `Fragment()`，可以使用 `FragmentFactory` 解决这个问题，这是在 `AndriodX Fragment 1.2.0` 中新增加的 API，详情可以查看另外一篇文章 [Google 建议使用这些 Fragment 的新特性](https://mp.weixin.qq.com/s/AVHkgYX8OVzbF2nxwQJ3qA)。

如果不使用 `AndriodX Fragment` 库，默认情况下系统是不支持的，虽然上面的代码可以正常编译运行，但是在运行过程当中，因为配置更改，导致在销毁恢复的过程中会崩溃，错误信息如下所示。

```kotlin
Caused by: java.lang.NoSuchMethodException: MainFragment.<init>
```

这是因为系统需要在某些情况下重新初始化它，比如配置更改，例如设备被旋转时，导致 Fragment 被销毁，如果没有默认空的构造函数，系统不知道如何重新初始化 Fragment 实例。


![](https://img.hi-dhl.com/202211131955514.jpeg)

因此，我们总是需要确保实例化 Fragment 的时候有一个空的构造函数。

### 面试题三：Instantiate ViewModel Directly

`ViewModel` 是 Jetpack 架构组件成员之一，花几秒钟思考一下，下面的代码会有什么问题？

```kotlin
class MainActivity: AppCompatActivity() {
    private val viewModel = MainViewModel()
}

class  MainViewModel(): ViewModel {

}
```


**错误回答**

1. 没有什么问题，可以正常编译运行
2. 我们还需要向它注入一些依赖项，例如 `MainViewModel (repository)`

**正确回答**

我们不应该直接实例化 `ViewModel`。 `ViewModel` 是 Jetpack 架构组件成员之一，意味着它可以在配置更改时存活，例如设备旋转时，它比 Activity 有更长的生命周期。

如果我们在代码中直接实例化 `ViewModel`，尽管它可以工作，但它将会变成一个普通的 `ViewModel`，失去原本拥有的特性。

因此，要实例化 `ViewModel`，建议使用 `ViewModel KTX`，代码如下所示。


```kotlin
class MainActivity: AppCompatActivity() {
    private val viewMode:MainViewModel by viewModels()
}
```

* `by viewModels ()` 会在重新启动或从已杀死的进程中恢复时，实例化一个新的 `ViewModel`。如果有配置更改，例如设备被旋转时，它将检查 `ViewModel` 是否已经创建，而不重新创建它

![](https://img.hi-dhl.com/202211131955515.jpeg)

* `ViewModels()` 会根据需要自动注入 `SavedInstancestate` （例如 Activity 中的 `SavedInstanceState` 和 `Intent`），如果我们有其他依赖是通过 `Dagger Hilt` 注入，它将与 `ViewModel` 一起使用下面的参数


```kotlin
@HiltViewModel
class MyViewModel @Inject constructor(
    private val repository: Repository,
    savedStateHandle: SavedStateHandle
) : ViewModel {
    
}
```

### 面试题四：ViewModel as StateRestoration Solution

**问题：**

Jetpack 架构组件提供的 `ViewModel` 的作用是什么？

```kotlin
class MainActivity: AppCompatActivity() {
    private val viewMode:MainViewModel by viewModels()
    // Some other Activity Code
}
```

**错误回答**

`ViewModel` 是用于状态恢复，例如当 `Activity` 被杀死并重新启动时，`ViewModel` 是用来帮助恢复到原始状态。

**正确回答**

`ViewModel` 实际上是 google 提供的 Jetpack 架构组件之一，它鼓励 Android 开发者使用 MVVM 设计模式。

它还有其它重要的功能，例如设备旋转时，即使 `Activity` 和 `Fragment` 被销毁，它们各自的 `ViewModel` 仍会保留，Google 在 `ViewModel` 中提供了一个名为 `savedStateHandle` 参数，该参数用于保存和恢复数据。


### 面试题五：LiveData as State Restoration Solution

**问题：**

Jetpack 架构组件提供的 `LiveData` 的作用是什么。

```kotlin
// Declaring it
val liveDataA = MutableLiveData<String>()
// Trigger the value change
liveDataA.value = someValue
```

**错误回答：**

它的存在是为了确保数据在 Activity 的生命周期中存活。当 Activity 在进程销毁返回时，数据将会自动恢复。

**正确回答**

`LiveData` 本身不能在进程销毁中存活。它是一种特殊类型的数据，根据观察者（Activity 或 Fragment）的生命周期来控制其发出的值。


![](https://img.hi-dhl.com/202211131955516.jpeg)

`ViewModel` 在配置变更后仍然存在，所以 `ViewModel` 内部的 `LiveData` 也一样。这确保 `LiveData` 发射值按照下图控制。

![](https://img.hi-dhl.com/202211131955517.jpeg)

然而 `LiveData` 本身不能在进程销毁中存活，当内存不足时，Activity 被系统杀死，`ViewModel` 本身也会被销毁。

为了解决这个问题，Google 在 `ViewModel` 中提供了一个名为 `savedStateHandle` 参数，该参数用于保存和恢复数据，以便数据在进程销毁后继续存在。

```kotlin
@HiltViewModel
class MyViewModel @Inject constructor(
    private val repository: Repository,
    savedStateHandle: SavedStateHandle
) : ViewModel {
    // Some other ViewModel Code
}
```


它是一种增强的机制，可以处理 `Intent` 和 `SavedInstanceState`，在以前的时候，这些都是由 `Activity` 单独处理的。

![](https://img.hi-dhl.com/202211131955518.jpeg)

为了确保 `Livedata` 保存下来，我们可以在 `SavedStateHandle` 中检查 `Livedata` 是否已经创建。

```kotlin
val liveData = savedStateHandle.getLiveData<String>(KEY)
```

类似地，这也适用于 `stateFlow`，它可以在进程销毁中存活下来。

```kotlin
val stateFlow = savedStateHandle.getStateFlow<String>(KEY, 0)
```

因此 `LiveData` 本身并不是用来恢复数据的。

### 面试题六：When is The View Destroyed But Not the Instance

**问题：**

在 Activity 或 Fragment 通常会有一个视图。你能给我提供一个场景，实例的视图被破坏了，但实例（例如 Activity 或 Fragment）还存在。

**错误回答**

1. 当配置发生变化时（例如设备旋转）
2. 当内存不足时，App 在后台运行，进程会杀死 App

**正确回答**

实际上 `Activity` 总是与其视图一起被销毁。因此，在 Activity 中没有 `onDestroyView ()` 生命周期方法。

只有在 `Fragment` 中有 `onDestroyView ()` 生命周期方法。在大多数情况下 `Fragment` 和它的视图一起被销毁。


但是通过 `Fragment transaction` 用一个 `Fragment` 替换另一个 `Fragment` 时，栈下面的 `Fragment` 仍然存在，但是它的视图被破坏了。

![](https://img.hi-dhl.com/202211131955519.jpeg)


当一个 `Fragment` 被另一个 `Fragment` 替换时，会调用 **onDestroyView ()** 方法，但不会调用 **onDestroy ()** 或 **onDetect ()** 方法。

正因为这种复杂性，在使用 `Fragment` 时，会遇到许多奇怪的问题。和 `Fragment` 相关的问题，将会在后面的文章中分享。

### 问题七：Lifecycle Aware Coroutine

在 App 中使用协程，如何确保它们的生命周期可感知。

**错误回答**


1. 对于普通视图，只需在 `Activity` 或 `Fragment` 中使用 `lifecycleScope`，在 `ViewModel` 中使用 `viewModelScope`
2. 对于组合视图，需要使用 `stateFlow` 中的 `collectAsState()` 方法，因为当可组合函数不活动时，它不会收集

**正确回答**

对于普通视图，即使 `lifecycleScope` 是可用的，它在 `Activity` 或 `Fragment` 的整个生命周期中都处于活动状态。因为有时我们希望某些场景只在 `onStart()` 或 `onResume()` 之后处于活动状态。

为此，我们需要在 `lifecycleScope` 中使用像 `repeatOnLifecycle` 这样的 API 提供额外的作用域。


```kotlin
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.stateFlowValue.collect {
            // Do something
        }
    }
}
```


它们的变体如下图所示。

![](https://img.hi-dhl.com/202211131955520.jpeg)


对于组合视图 `collectAsState()` 不会确保在组合函数处于活动状态时安全使用数据，它也不会停止继续发送 `StateFlow`，这会导致资源浪费。

为了确保只在 `Activity` 或 `Fragment` 处于正确的生命周期时，例如在 `onStart ()` 之后发出，我们需要使用 `collectAsStateWithLifecycle ()` 和 `stateFlow` 中的 `WhileSubscribed (...)`。

当我们在研究 `collectAsStateWithLifecycle()` 源码时，发现它也在使用  `repeatOnLifecycle(…)`。


![](https://img.hi-dhl.com/202211131955521.jpeg)


<br/>

**全文到这里就结束了，感谢你的阅读，坚持原创不易，欢迎在看、点赞、分享给身边的小伙伴，我会持续分享原创干货！！！**

___

我开了一个云同步编译工具（SyncKit），主要用于本地写代码，同步到远程设备，在远程设备上进行编译，最后将编译的结果同步到本地，代码已经上传到 Github，欢迎前往仓库 [hi-dhl/SyncKit](https://github.com/hi-dhl/SyncKit)  查看。

* [仓库 SyncKit：https://github.com/hi-dhl/SyncKit](https://github.com/hi-dhl/SyncKit)
* [下载地址：https://github.com/hi-dhl/SyncKit/releases](https://github.com/hi-dhl/SyncKit/releases)

___

**真诚推荐你关注我，公众号：ByteCode ，持续分享硬核原创内容，Kotlin、Jetpack、性能优化、系统源码、算法及数据结构、动画、大厂面经。**

<div align="center">
<table>
<tr>
<td><a href=" https://mp.weixin.qq.com/mp/homepage?__biz=MzAwNDgwMzU4Mw==&hid=9&sn=e90937096f96a747c08e27ab7a79545c&scene=1&devicetype=android-30&version=28001c57&lang=zh_CN&nettype=WIFI&ascene=7&session_us=gh_d4a2dd00574d&wx_header=3">公众号 ：ByteCode</a></td>
 <td><a href=" https://space.bilibili.com/498153238">哔哩哔哩</a></td>
 <td><a href=" https://juejin.im/user/2594503168898744">掘金</a></td>
 <td><a href=" https://hi-dhl.com">博客</td>
 <td><a href=" https://github.com/hi-dhl">Github</a></td>

 </tr>
 </table>
</div>

___

**最新文章**

* [Android 13这些权限废弃，你的应用受影响了吗？](https://mp.weixin.qq.com/s/t_E6kOU2jTJ82pcJ7ZkdMA)
* [Android 12 已来，你的 App 崩溃了吗？](https://mp.weixin.qq.com/s/NuqAYoUq_0OorM1rVHUEHA)
* [Android 利器，我开发了云同步编译工具](https://mp.weixin.qq.com/s/jRlrtnOg6Ww-C_Xb6719EA)
* [Twitter 上有趣的代码](https://mp.weixin.qq.com/s/ExxJMyYZP3sd9pnvdiEFAg)
* [谁动了我的内存，揭秘 OOM 崩溃下降 90% 的秘密](https://mp.weixin.qq.com/s/QZ6zZ6GoNpB5MWGi31k-UA)
* [反射技巧让你的性能提升 N 倍](https://mp.weixin.qq.com/s/AgEr6GhylkUfG_zWE5LTOw)
* [90%人不懂的泛型局限性，泛型擦除，星投影](https://mp.weixin.qq.com/s/brNq5OxitQ7WhPbA9y5lrA)
* [揭秘反射真的很耗时吗，射 10 万次耗时多久](https://mp.weixin.qq.com/s/Ah8Yau_UW07s6LnGjrG4hA)
* [Google 宣布废弃 LiveData.observe 方法](https://mp.weixin.qq.com/s/fp1ZOmqAcEBv2f7ec1r-zw)
* [影响性能的 Kotlin 代码（一）](https://mp.weixin.qq.com/s/8dAbt1-mcCVLWLXKC-1_xw)
* [揭秘 Kotlin 中的 == 和 ===](https://mp.weixin.qq.com/s/sYj_-wqENr9Jaw1p8iP4Jg)


**开源新项目**


* 云同步编译工具（SyncKit），本地写代码，远程编译，欢迎前去查看 [SyncKit](https://github.com/hi-dhl/SyncKit)

* KtKit 小巧而实用，用 Kotlin 语言编写的工具库，欢迎前去查看 [KtKit](https://github.com/hi-dhl/KtKit)

* 最全、最新的 AndroidX Jetpack 相关组件的实战项目以及相关组件原理分析文章，正在逐渐增加 Jetpack 新成员，仓库持续更新，欢迎前去查看 [AndroidX-Jetpack-Practice](https://github.com/hi-dhl/AndroidX-Jetpack-Practice)

* LeetCode / 剑指 offer，包含多种解题思路、时间复杂度、空间复杂度分析，[在线阅读](https://leetcode.hi-dhl.com)



