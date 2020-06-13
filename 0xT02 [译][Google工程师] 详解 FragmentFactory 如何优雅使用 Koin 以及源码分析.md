# 0xT02 [译][Google工程师] 详解 FragmentFactory 如何优雅使用 Koin 以及源码分析

![](http://cdn.51git.cn/2020-05-23-15901559116328.jpg)

## 前言

> * 原标题:  Android Fragments: FragmentFactory
> * 原文地址: [https://proandroiddev.com/fragmentfactory......](https://proandroiddev.com/android-fragments-fragmentfactory-ceec3cf7c959)
> * 原文作者：Husayn Hakeem

在之前的文章 [[译][Google工程师] 刚刚发布了 Fragment 的新特性 “Fragment 间传递数据的新方式” 以及源码分析](https://juejin.im/post/5eb58da05188256d6d6bb248) 介绍了 Fragment 1.3.0 中添加了几个重要的 API。

继续上一篇文章，介绍一下 FragmentFactory 和 FragmentContainerView 以及如何和 Koin 一起使用， 这是 Google 在 Fragment 1.2.0 上做的重要的更新，强烈建议大家去使用

**通过这篇文章你将学习到以下内容，将在译者思考部分会给出相应的答案**

* FragmentFactory 是什么？
* 什么情况下使用 FragmentFactory？
* FragmentContainerView 是什么？
* 为什么 Google 强烈建议使用 FragmentContainerView？
* Koin 如何和 FragmentFactory 一起使用以及源码分析？
* 如何处理嵌套 Fragment?

这篇文章涉及很多重要新的知识点，带着自己理解，请耐心读下去，应该可以从中学到很多技巧。

## 译文

现在我们可以使用 FragmentFactory 来完成 Fragment 构造函数的注入，但是这不是开发人员必须使用的 API, 在某些情况下，它可以被认为是一种很好的设计方法，帮助我们测试带有外部依赖项的 Fragment。

这篇文章将会解释什么是 FragmentFactory，什么时候以及如何使用它，如何处理嵌套 Fragment。

### 什么是 FragmentFactory？

之前 Fragment 的实例都是通过使用默认的空的构造函数进行实例化，这是因为系统需要在某些情况下重新初始化它，比如配置更改或者 App 的进程重建，如果没有默认构造函数的限制，系统不知道如何重新初始化 Fragment 实例。

FragmentFactory 出现就是为了解决这个限制，通过向其提供实例化 Fragment 所需的参数/依赖关系，FragmentFactory 可以帮助系统创建 Fragment 实例。

### 如何使用 FragmentFactory？

如果你的 Fragment 没有空的构造函数，您需要创建一个 FragmentFactory 来处理初始化它，通过继承 FragmentFactory 并且覆盖 [FragmentFactory#instantiate()](https://developer.android.com/reference/androidx/fragment/app/FragmentFactory.html#instantiate(java.lang.ClassLoader,%20java.lang.String)) 来完成。

```
class CustomFragmentFactory(private val dependency: Dependency) : FragmentFactory() {
    override fun instantiate(classLoader: ClassLoader, className: String): Fragment {
        if (className == CustomFragment::class.java.name) {
            return CustomFragment(dependency)
        }
        return super.instantiate(classLoader, className)
    }
}
```

Fragment 是由 FragmentManagers 来管理的，所以为了使用 FragmentFactory 需要关联 FragmentManager，更具体的说它必须分配给包含 Fragment 组件的 FragmentManager，它可以是 Activity 或者 Fragment。

### 什么时候 FragmentFactory 和 FragmentManager 做关联

FragmentFactory 负责在 Activity 和 parent Fragment 初始化 Fragment，所以应该在创建 Fragment 之前设置它。

* 在创建 component’s View 之前：如果在 XML 中定义 Fragment，应该使用 Fragment 的 tag `<fragment>` 或者 FragmentContainerView。
* 在创建 Fragment 之前：如果 Fragment 是动态添加的应该使用 FragmentTransaction。
* 在系统恢复 Fragment 之前：如果是因为配置更改或者 App 的进程重建，导致 Fragment 重建。

有了这些限制，可以在 [Activity#onCreate()](https://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle,%20android.os.PersistableBundle)) 和 [Fragment#onCreate()](https://developer.android.com/reference/androidx/fragment/app/Fragment.html#onCreate(android.os.Bundle)) 之前关联 FragmentFactory 和 FragmentManager，在这两个调用处 view 创建之前会重新初始化 Fragment。

这也就意味着应该在 super#onCreate() 之前关联 FragmentFactory 和 FragmentManager。

* 在 Activity 关联 FragmentFactory 和 FragmentManager

```
class HostActivity : AppCompatActivity() {
    private val customFragmentFactory = CustomFragmentFactory(Dependency())

    override fun onCreate(savedInstanceState: Bundle?) {
        supportFragmentManager.fragmentFactory = customFragmentFactory
        super.onCreate(savedInstanceState)
        // ...
    }
}
```

* 在 Fragment 关联 FragmentFactory 和 FragmentManager

```
class ParentFragment : Fragment() {
    private val customFragmentFactory = CustomFragmentFactory(Dependency())

    override fun onCreate(savedInstanceState: Bundle?) {
        childFragmentManager.fragmentFactory = customFragmentFactory
        super.onCreate(savedInstanceState)
        // ...
    }
}
```

### 需要使用 FragmentFactory 吗？


到目前为止，您可能已经使用它们的默认构造函数创建 Fragment，然后使用 Dagger 或 Koin 这样的库注入它们需要的依赖项，或者在它们被使用之前在 Fragment 中初始化它们。

如果你的 Fragment 有一个默认的空构造函数，那么就不需要使用 FragmentFactory，如果在 Fragment 构造函数中接受参数，必须使用 FragmentFactory，否者会抛出 [Fragment.InstantiationException](https://developer.android.com/reference/kotlin/androidx/fragment/app/Fragment.InstantiationException.html) 异常 

### 如何同时使用 Fragment 和 FragmentFactory？

只需要在创建 Fragment 之前，设置 FragmentFactory，它就会被用来实例化，这意味着在添加 Fragments 之前使用自定义的 FragmentFactory。

* **静态添加：** 使用 Fragment 的 tag `<fragment>` 和 FragmentContainerView。

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/customFragment"
    android:name="com.example.CustomFragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:tag="custom_fragment" />
```

设置 FragmentFactory 用于初始化在Fragment 声明的 FragmentContainerView

```
class HostActivity : AppCompatActivity() {
    private val customFragmentFactory = CustomFragmentFactory(Dependency())

    override fun onCreate(savedInstanceState: Bundle?) {
        supportFragmentManager.fragmentFactory = customFragmentFactory
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_with_fragment_container_view)
    }
}
```

* **动态添加：** 使用 FragmentTransaction#add() 方法动态的添加 Fragment

```
class HostActivity : AppCompatActivity() {
    private val customFragmentFactory = CustomFragmentFactory(Dependency())

    override fun onCreate(savedInstanceState: Bundle?) {
        supportFragmentManager.fragmentFactory = customFragmentFactory
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_with_empty_frame_layout)
        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .add(R.id.content, CustomFragment::class.java, arguments)
                .commit()
        }
    }
}
```

### FragmentFactory 和嵌套的 Fragment

如果 parent Fragment 包含嵌套的 Fragment 或者多层次嵌套的 Fragment，它们都会使用 parent Fragment 的相同 FragmentFactory，嵌套 Fragment 需要调用 Fragment#childFragmentManager.fragmentFactory

```
class ParentFragment : Fragment() {
    override fun onCreate(savedInstanceState: Bundle?) {
        childFragmentManager.fragmentFactory = parentFragmentFactory
        super.onCreate(savedInstanceState)
        if (savedInstanceState == null) {
            // Add NestedFragment
        }
    }
}

class NestedFragment : Fragment() {
    override fun onCreate(savedInstanceState: Bundle?) {
        childFragmentManager.fragmentFactory = childFragmentFactory
        super.onCreate(savedInstanceState)
        if (savedInstanceState == null) {
            // Add NestedNestedFragment
        }
    }
}

class NestedNestedFragment : Fragment()
```

## 译者思考

我们来总结一下 Fragment 几个重要的更新，以及在什么情况下使用：

* 之前 Fragment 的实例都是通过使用默认的空的构造函数进行实例化的，FragmentFactory 出现就是为了解决这个限制。
* FragmentFactory 不是必须要使用的，如果在 Fragment 构造函数中接受参数，必须使用 FragmentFactory
* FragmentFactory 需要在 Activity 或者 Fragment 中使用，并且需要在 [Activity#onCreate()](https://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle,%20android.os.PersistableBundle)) 和 [Fragment#onCreate()](https://developer.android.com/reference/androidx/fragment/app/Fragment.html#onCreate(android.os.Bundle)) 之前和 FragmentManager 做关联
* 嵌套的 Fragment 或者多层次嵌套的 Fragment，使用的是相同 FragmentFactory
* 正因为 FragmentFactory 出现，可以在 Fragment 构造函数中传递参数，意味着可以使用 Koin 等框架，可以实现构造函数依赖注入，后面我会演示如何使用

接下来一起了解一下什么 FragmentContainerView，为什么 Google 强烈建议使用 FragmentContainerView 容器来存储动态添加的 Fragment。

### FragmentContainerView 是什么？为什么 Google 强烈建议使用？

我们先来看一下 Google 的更新说明：

![](http://cdn.51git.cn/2020-05-23-15902004126751.jpg)

FragmentContainerView： FragmentContainerView 是一个自定义 View 继承 FrameLayout，与 ViewGroups 不同，它只接受 Fragment Views。

**为什么 Google 强烈建议使用？**

之前在 [Google issue](https://b.corp.google.com/issues/37036000) 提了一个 fragment z-ordering 的问题，就是说 Fragment 进入和退出动画会导致一个问题，进入的 Fragment 会在退出的 Fragment下面，直到它完全退出屏幕，这会导致在 Fragment 之间切换时产生错误的动画。

使用 FragmentContainerView 带来的好处是改进了对 fragment z-ordering 的处理。这是 Google 演示的[例子](https://www.youtube.com/watch?v=RS1IACnZLy4&feature=youtu.be&t=548)，优化了两个 Fragment 退出和进入过渡不会互相重叠，使用 FragmentContainerView 将先开启退出动画然后才是进入动画。

### Koin 如何和 FragmentFactory 一起使用以及源码分析

在之前的文章 [[译][2.4K Start] 放弃 Dagger 拥抱 Koin](https://juejin.im/post/5ebc1eb8e51d454dcf45744e?utm_source=gold_browser_extension) 分析了 Koin 性能，如果没有看过，建议可以去了解一下。

Koin 团队在 2.1.0 版本开始支持 Fragment 的依赖注入，截图如下所示：

![](http://cdn.51git.cn/2020-05-23-15901552967574.jpg)

**1. 添加 Koin Fragment 依赖**

```
implementation "org.koin:koin-androidx-fragment:2.1.5"
```

**2. 创建 Fragment 并传递 ViewModel**

```
class FragmentTest(val mainViewModel: MainViewModel) : Fragment(){
    ......
}
```

**3. 创建 Fragment modules**

```
val viewModelsModule = module {
    viewModel { MainViewModel() }
}

val fragmentModules = module {
    fragment { FragmentTest(get()) }
}

val appModules = listOf(fragmentModules, viewModelsModule)
```

**4. 在调用 startKoin 方法时设置 KoinFragmentFactory**

```
startKoin {
    AndroidLogger(Level.DEBUG)
    androidContext(this@App)
    fragmentFactory()
    loadKoinModules(appModules)
}
```

fragmentFactory 是 KoinApplication 的扩展函数，提供了  KoinFragmentFactory 代码如下所示：

```
koin.loadModules(listOf(module {
        single<FragmentFactory> { KoinFragmentFactory() }
    }))
}
```

一起来分析 KoinFragmentFactory 内部的源码：

```
class KoinFragmentFactory(val scope: Scope? = null) : FragmentFactory(), KoinComponent {

    override fun instantiate(classLoader: ClassLoader, className: String): Fragment {
        val clazz = Class.forName(className).kotlin
        val instance = if (scope != null) {
            scope.getOrNull<Fragment>(clazz)
        }else{
            getKoin().getOrNull<Fragment>(clazz)
        }
        return instance ?: super.instantiate(classLoader, className)
    }

}
```

继承 FragmentFactory 并且重写了 FragmentFactory#instantiate() 方法，在这个函数中，我们使用 className 作为参数获取 Fragment，并尝试从 Koin 中检索 Fragment 实例

**5. 在 onCreate 方法之前 调用 setupKoinFragmentFactory 绑定 FragmentFactory**

```
override fun onCreate(savedInstanceState: Bundle?) {
    setupKoinFragmentFactory()
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
}
```

**6. 添加 Fragment 并传递 Bundle**

```
val arguments = Bundle().apply {
    putString(FragmentTest.KEY_NAME, "来源于 MainActivity")
}

supportFragmentManager.beginTransaction()
    .replace(R.id.container, FragmentTest::class.java, arguments)
    .commit()
```

相关源码已经上传到 [JDataBinding](https://github.com/hi-dhl/JDataBinding) 中, 可以查看 App、MainActivity、AppModule 和 FragmentTest 这几个类

## 参考文献

* [https://github.com/InsertKoinIO/koin/issues/647](https://github.com/InsertKoinIO/koin/issues/647)
* [Android Fragments: FragmentFactory](https://proandroiddev.com/android-fragments-fragmentfactory-ceec3cf7c959)
* [https://developer.android.com/jetpack/androidx/releases/fragment](https://developer.android.com/jetpack/androidx/releases/fragment)

## 结语

致力于分享一系列 Android 系统源码、逆向分析、算法、翻译相关的文章，目前正在翻译一系列欧美精选文章，请持续关注，除了翻译还有对每篇欧美文章思考，如果对你有帮助，请帮我点个赞，感谢！！！期待与你一起成长。

