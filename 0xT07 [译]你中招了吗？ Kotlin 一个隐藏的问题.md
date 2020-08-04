# 0xT07 [译]你中招了吗？ Kotlin 一个隐藏的问题

![](http://cdn.51git.cn/2020-08-04-15964757436916.jpg)

## 前言

> * 原标题:  A decompiled story of Kotlin let and run
> * 原文地址: [https://dev.to/vlazdra/a-decompiled......](https://dev.to/vlazdra/a-decompiled-story-of-kotlin-let-and-run-4k83)
> * 原文作者：Vladimir Zdravkovic

之前我发表过几篇关于 Kotlin 性能损耗和 Kotlin 技巧方面的文章，如果没有了解过，可以点击下方链接前去查看，可以避免在实际开发中遇到重复的问题。

* [[译][2.4K Start] 放弃 Dagger 拥抱 Koin](https://juejin.im/post/5ebc1eb8e51d454dcf45744e?utm_source=gold_browser_extension)
* [[译][5k+] Kotlin 的性能优化那些事](https://juejin.im/post/5ec0f3afe51d454db11f8a94#heading-7)
* [为数不多的人知道的 Kotlin 技巧以及 原理解析(一)](https://juejin.im/post/5edfd7c9e51d45789a7f206d)
* [为数不多的人知道的 Kotlin 技巧以及 原理解析(二)](https://juejin.im/post/6847902224467623950)

这篇文章主要来分析 Kotlin 另外一个隐藏的问题，文章将会分为 **译文** 和 **译者思考** 两部分，本文不仅仅是翻译，在 **译者思考** 部分会对译文进行总结以及更加深入的思考和分析，也可以跳过译文直接看 **译者思考** 部分。

**通过这篇文章你将学习到以下内容，将在译者思考部分会给出相应的答案**

* 使用 `T.let` 会遇到什么问题？
* 为什么会造成这个问题？
* 如何解决这个问题？
* 为什么使用 `T.apply` 可以解决这个问题？
* `T.apply` 和 `T.let` 有什么区别？
* 如何区分 run, with, let, also, apply？
* Kotlin 如何交换两个变量？

这篇文章涉及很多重要的知识点，请耐心读下去，应该可以从中学到很多技巧。

## 译文

前段时间，我花了不少时间，为了弄清楚一个简单的 Kotlin 代码块 `let` 和 `run`，为什么不想做我想让它做的事，经过无数次重写我的代码之后，我认为这一定是我自己的错误，最后为了搞清楚 Kotlin 语法糖背后发生了什么，最后我决定花点时间深入研究生成的代码。

`let` 和 `run` 是 Kotlin 标准库当中的内联函数，我认为它们的工作的方式与经典的 `if ... else ...` 语句类似，所以我一直在项目中这么使用它们，直到我在一个项目中为了实现某个功能的时候，遇到了一个隐藏的问题，让我们看看问题是什么。

![](http://cdn.51git.cn/2020-08-04-159629675122671.jpg)

这是一个很简单的 Kotlin 代码，它有两个可空的变量，其中一个已经有值，如果我调用 `doSomeAwesomePrinting()` 方法，你认为控制台会输出什么？

你可能会和我一样认为什么都不会输出，可是... 错了，最后会输出 "awesome output 1"。

为什么会这样？我们来看一下反编译后的代码，发生了什么。

![](http://cdn.51git.cn/2020-08-04-159634085308213.jpg)



正如你所见，当第二个变量 awesomeVar2 为空时，Kotlin 自动生成的变量 `var10000` 也为空，所以程序不会执行 return 语句，函数执行到最后会输出 ”awesome output 1“。

让我们来看看另外一个例子，在这个例子中，我们对上面 Kotlin 代码做一些更改，在第二个变量上添加 elvis 操作符，代码如下所示。

![](http://cdn.51git.cn/2020-08-04-159629754266153.jpg)


如果再次调用 `doSomeAwesomePrinting()` 方法则会输出 "awesome output 3"，这次的修改已完成了想要做的事情，与经典的 `if ... else ...` 语句类似，我们来看一下反编译后的代码。

![](http://cdn.51git.cn/2020-08-04-159634121091672.jpg)


正如你所见，反编译后的代码其实就是 `if ... else ...` 语句，当第二个变量为空时则输出 "awesome output 3"， 现在来分析一下如何解决文章开头提出来的问题。

**解决方案**

感谢 Danny 的建议，其实可以用 Kotlin 另外的一个内联函数 apply 来解决这个问题。

![](http://cdn.51git.cn/2020-08-04-15962985306741.jpg)

接下来按照 Danny 的建议，使用 Kotlin 另外一个内联函数 apply，看看会发生什么。

![](http://cdn.51git.cn/2020-08-04-159629868566451.jpg)


和我们所期望的一样，当第二个变量为空时，控制台什么都不会输出，你可以试一下，接下来我们来分析一下反编译后的代码。

![](http://cdn.51git.cn/2020-08-04-159629876789301.jpg)

正如你所看到的，Kotlin 自动生成的变量 var1 不为空，当第二个变量 awesomeVar2 为空时，直接 return 了。

最后我们对上面 Kotlin 代码，在 apply 的基础上，做一点一点修改，在第二个变量上添加 elvis 操作符如下所示。

![](http://cdn.51git.cn/2020-08-04-159629901151061.jpg)

运行代码之后，当第二个变量为空时，控制台将会输出 “awesome output 3” ，为了能够理解这里发生了什么，我们来看一下反编译后的代码。

![](http://cdn.51git.cn/2020-08-04-159629927356662.jpg)

生成的代码比之前多了很多，但是不影响我们正常分析，和我们预期的一样，控制台将输出 “awesome output 3”

## 译者思考

接下来是译者思考部分，按照之前的风格，我们先对译文进行总结，然后在进行分析。

### 总结和分析

**使用 T.let 会遇到了什么问题？**

Kotlin 标准库当中的内联函数 T.let 和 T.run 等等，它们的工作的方式与 `if ... else ...` 语句类似，从反编译后的代码可知其实就是 `if ... else ...` 语句，所以我们可能会认为运行下面的代码和 `if ... else ...` 语句一样不会有任何输出。

```
class ExampleClass {
    var awesomeVar1: String? = "some awesome string value"
    var awesomeVar2: String? = null

    fun doSomeAwesomePrinting() {
        awesomeVar1?.let {
            awesomeVar2?.let {
                println("awesome output 2")
            }
        } ?: run {
            println("awesome output 1")
        }
    }
}
```

但是结果却出人意料，当第二个变量为空时，居然输出 "awesome output 1"，大家可以反编译上面的代码看一下，会更加清楚其内部逻辑。

**为什么会造成这个问题？**

我们来看一下 Kotlin 内联函数 T.let 的源码实现。

```
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}
```

正如你所看到的 T.let 接受的参数是 it, 它返回最后一行，接下来我们将源代码拆解一下，可能会更清楚发生了什么。

```
fun doSomeAwesomePrinting() {
    awesomeVar1?.let {

        // 将 awesomeVar2?.let{...}  的结果作为 awesomeVar1?.let{...} 的返回值
        // 所以当 awesomeVar2 为空时，awesomeVar2?.let{...} 的结果为空
        // 函数最后会输出 "awesome output 1"
        awesomeVar2?.let { println("awesome output 2") }

    } ?: run {
        println("awesome output 1")
    }
}
```

* 将 `awesomeVar2?.let{...}` 的结果作为 `awesomeVar1?.let{...}` 的返回值
* 当 `awesomeVar2` 为空时，`awesomeVar2?.let{...}` 的结果为空
* 函数最后会输出 "awesome output 1"

**如何解决这个问题？**

解决方案也很简单使用另外一个 Kotlin 内联函数 T.apply，代码如下所示。

```
class ExampleClass {
    var awesomeVar1: String? = "some awesome string value"
    var awesomeVar2: String? = null

    fun doSomeAwesomePrinting() {
        awesomeVar1?.apply {
            awesomeVar2?.apply {
                println("awesome output 2")
            }
        } ?: run {
            println("awesome output 1")
        }
    }
}
```

使用 Kotlin 另外一个内联函数 T.apply 之后，结果和我们所预期的一样，这里什么都不会输出，那么为什么使用 T.apply 可以解决这个问题？T.apply 和 T.let 有什么区别呢？

**为什么使用 T.apply 可以解决这个问题？**

我们来看一下 Kotlin 内联函数 T.apply 的源码实现。

```
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

T.apply 函数是一个扩展函数，返回值是它本身，并且接受的参数是 this，接下来我们将源代码拆解进行分析。

```
fun doSomeAwesomePrinting() {
    awesomeVar1?.apply {

        // awesomeVar1?.apply{...} 的返回值是它本身，awesomeVar1 不为空
        // 所以当 awesomeVar2 为空时，什么都不会输出
        awesomeVar2?.apply { println("awesome output 2") } // awesomeVar2?.apply{...} 返回值是 awesomeVar2

    } ?: run {
        println("awesome output 1")
    }
}
```

* 根据 T.apply 函数特性，`awesomeVar1?.apply{...}` 的返回值是它本身，变量 `awesomeVar1`不为空
* 当 `awesomeVar2` 为空时，并不会影响到 `awesomeVar1?.apply{...}` 的结果，所这里什么都不会输出

**T.apply 和 T.let 有什么区别？**

| 函数 | 是否是扩展函数 | 函数参数(this、it) | 返回值(调用本身、最后一行) |
| --- | --- | --- | --- |
| T.let | 是 | it | 最后一行 |
| T.apply | 是 | this | 调用本身 |


除了 T.apply 和 T.let 之外，Kotlin 还有很多其他内联函数例如 T.run 、T.also、with 等等，虽然操作符不多，但是想要分清楚这些操作符确实有些困难，所以我将会介绍一种简单的方法来区分它们以及如何使用。

### 如何区分 run, with, let, also, apply

感谢大神 Elye 的这篇文章提供的思路 [Mastering Kotlin standard functions](https://medium.com/@elye.project/mastering-kotlin-standard-functions-run-with-let-also-and-apply-9cd334b0ef84)。

关于如何区分 run, with, let, also, apply 我在之前的文章  [为数不多的人知道的 Kotlin 技巧以及 原理解析](https://juejin.im/post/5edfd7c9e51d45789a7f206d) 中有介绍，这里我们在回顾一下。

run, with, let, also, apply 是 Kotlin 的内联函数，也是作用域函数，这些作用域函数如何使用，以及如何区分呢，我们将从以下三个维度来区分它们。

* 是否是扩展函数。
* 作用域函数的参数（this、it）。
* 作用域函数的返回值（调用本身、其他类型即最后一行）。

### 是否是扩展函数

首先我们来看一下 with 和 T.run，这两个函数非常的相似，他们的区别在于 with 是个普通函数，T.run 是个扩展函数，来看一下下面的例子。

```
val name: String? = null
with(name){
    val subName = name!!.substring(1,2)
}

// 使用之前可以检查它的可空性
name?.run { val subName = name.substring(1,2) }?:throw IllegalArgumentException("name must not be null")
```

在这个例子当中，name?.run 会更好一些，因为在使用之前可以检查它的可空性。

### 作用域函数的参数（this、it）

我们在来看一下 T.run 和 T.let，它们都是扩展函数，但是他们的参数不一样，T.run 的参数是 this, T.let 的参数是 it。

```
val name: String? = "hi-dhl.com"

// 参数是 this，可以省略不写
name?.run {
    println("The length  is ${this.length}  this 是可以省略的 ${length}")
}

// 参数 it
name?.let {
    println("The length  is  ${it.length}")
}

// 自定义参数名字
name?.let { str ->
    println("The length  is  ${str.length}")
}
```

在上面的例子中似乎 T.run 会更好，因为 this 可以省略，调用更加的简洁，但是 T.let 允许我们自定义参数名字，使可读性更强，如果倾向可读性可以选择 T.let。

### 作用域函数的返回值（调用本身、其他类型即最后一行）

接下里我们来看一下 T.let 和 T.also 它们接受的参数都是 it, 但是它们的返回值是不同的，T.let 返回的是最后一行，T.also 返回调用者本身。

```
var name = "hi-dhl"

// 返回调用本身
name = name.also {
    val result = 1 * 1
    "juejin"
}
println("name = ${name}") // name = hi-dhl

// 返回的最后一行
name = name.let {
    val result = 1 * 1
    "hi-dhl.com"
}
println("name = ${name}") // name = hi-dhl.com
```

从上面的例子来看 T.also 似乎没有什么意义，细想一下其实是非常有意义的，在使用之前可以进行自我操作，结合其他的函数，功能会更强大。


```
fun makeDir(path: String) = path.let{ File(it) }.also{ it.mkdirs() }
```

当然 T.also 还可以做其他事情，比如利用 T.also 在使用之前可以进行自我操作特点，可以实现一行代码交换两个变量，在后面会有详细介绍

### T.apply 函数

通过上面的分析，大致了解了函数的行为，接下来看一下 T.apply 函数，T.apply 函数是一个扩展函数，返回值是它本身，并且接受的参数是 this。

```
// 普通方法
fun createInstance(args: Bundle) : MyFragment {
    val fragment = MyFragment()
    fragment.arguments = args
    return fragment
}
// 改进方法
fun createInstance(args: Bundle) 
          = MyFragment().apply { arguments = args }
          
          
// 普通方法
fun createIntent(intentData: String, intentAction: String): Intent {
    val intent = Intent()
    intent.action = intentAction
    intent.data=Uri.parse(intentData)
    return intent
}
// 改进方法，链式调用
fun createIntent(intentData: String, intentAction: String) =
    Intent().apply { action = intentAction }
            .apply { data = Uri.parse(intentData) }
```

### 使用 T.also 函数交换两个变量

接下来演示的是使用 T.also 函数，实现一行代码交换两个变量？我们先来回顾一下 Java 的做法。

```
int a = 1;
int b = 2;

// Java - 中间变量
int temp = a;
a = b;
b = temp;
System.out.println("a = "+a +" b = "+b); // a = 2 b = 1

// Java - 加减运算
a = a + b;
b = a - b;
a = a - b;
System.out.println("a = " + a + " b = " + b); // a = 2 b = 1
    
// Java - 位运算
a = a ^ b;
b = a ^ b;
a = a ^ b;
System.out.println("a = " + a + " b = " + b); // a = 2 b = 1

// Kotlin
a = b.also { b = a }
println("a = ${a} b = ${b}") // a = 2 b = 1

```

一起来分析 T.also 是如何做到的，其实这里用到了 T.also 函数的两个特点。

* 调用 T.also 函数返回的是调用者本身。
* 在使用之前可以进行自我操作。

也就是说 b.also { b = a } 会先将 a 的值 (1) 赋值给 b，此时 b 的值为 1，然后将 b 原始的值（2）赋值给 a，此时 a 的值为 2，实现交换两个变量的目的。

### 汇总

为了更方便的理解和记忆，接下来我以表格的形式将上面的内容进行汇总，具体还需要有结合实际项目去使用。

| 函数 | 是否是扩展函数 | 函数参数(this、it) | 返回值(调用本身、最后一行) |
| --- | --- | --- | --- |
| with | 不是 | this | 最后一行 |
| T.run | 是 | this | 最后一行 |
| T.let | 是 | it | 最后一行 |
| T.also | 是 | it | 调用本身 |
| T.apply | 是 | this | 调用本身 |

全文到这里就结束了，大家可以在项目中灵活的去运用，可以让代码的可读性更高，如果你在 Kotlin 中遇到了那些坑，欢迎在评论区分享，更多优秀的英文技术文章，点击这里 [精选译文](https://github.com/hi-dhl/Technical-Article-Translation)

更多关于 Kotlin 的内联函数 run, with, let, also, apply 在实际项目中的使用，可以看我另外一个项目 PokemonGo ，基于 Jetpack + MVVM + Repository + Data Mapper + Kotlin Flow + Kotlin 技巧的实战项目，可以点击下面链接前去查看。

[PokemonGo 仓库地址：https://github.com/hi-dhl/PokemonGo](https://github.com/hi-dhl/PokemonGo)

![PokemonGo](http://cdn.51git.cn/2020-07-23-Pokemon12.png)

## 结语

致力于分享一系列 Android 系统源码、逆向分析、算法、翻译、Jetpack 源码相关的文章，正在努力写出更好的文章，如果这篇文章对你有帮助给个 star，文章中有什么没有写明白的地方，或者有什么更好的建议欢迎留言，欢迎一起来学习，在技术的道路上一起前进。

> 计划建立一个最全、最新的 AndroidX Jetpack 相关组件的实战项目 以及 相关组件原理分析文章，正在逐渐增加 Jetpack 新成员，仓库持续更新，可以前去查看：[AndroidX-Jetpack-Practice](https://github.com/hi-dhl/AndroidX-Jetpack-Practice), 如果这个仓库对你有帮助，请帮我点个赞，我会陆续完成更多 Jetpack 新成员的项目实践。

### Android 应用系列

* [Jetpack 最新成员 AndroidX App Startup 实践以及原理分析](https://juejin.im/post/5ee4bbe4f265da76b559bdfe)
* [Jetpack 成员 Paging3 实践以及源码分析（一）](https://juejin.im/post/5ee998e8e51d4573d65df02b)
* [Jetpack 新成员 Paging3 网络实践及原理分析（二）](https://juejin.im/post/5eeefbf4e51d45742c53ddce)
* [Jetpack 成员 Paging3 使用 RemoteMediator 实现加载网络分页数据并更新到数据库中（三）](https://juejin.im/post/6854573220457086990#heading-1)
* [Jetpack 新成员 Hilt 实践（一）启程过坑记](https://juejin.im/post/5ef2f31951882565a94e06a5?utm_source=gold_browser_extension) 
* [Jetpack 新成员 Hilt 实践之 App Startup（二）进阶篇](https://juejin.im/post/5ef7638c5188252e6a532db3)
* [Jetpack 新成员 Hilt 与 Dagger 大不同（三）落地篇](https://juejin.im/post/5efca0c1e51d4534a40d972f)
* [全方面分析 Hilt 和 Koin 性能](https://juejin.im/post/5f02114d5188252e8a081afb)
* [神奇宝贝(PokemonGo)  眼前一亮的 Jetpack + MVVM 极简实战](https://juejin.im/post/5f0d303e6fb9a07e76550d4c?utm_source=gold_browser_extension) 
* [Google 推荐在 MVVM 架构中使用 Kotlin Flow](https://juejin.im/post/5f153adff265da22fb287e6e)
* [Google 推荐在项目中使用 sealed 和 RemoteMediator](https://juejin.im/post/6854573220457086990)
* [为数不多的人知道的 Kotlin 技巧以及 原理解析(一)](https://juejin.im/post/5edfd7c9e51d45789a7f206d)
* [为数不多的人知道的 Kotlin 技巧以及 原理解析(二)](https://juejin.im/post/6847902224467623950)

### 算法

由于 LeetCode 的题库庞大，每个分类都能筛选出数百道题，由于每个人的精力有限，不可能刷完所有题目，因此我按照经典类型题目去分类、和题目的难易程度去排序。

* 数据结构： 数组、栈、队列、字符串、链表、树……
* 算法： 查找算法、搜索算法、位运算、排序、数学、……

每道题目都会用 Java 和 kotlin 去实现，并且每道题目都有解题思路、时间复杂度和空间复杂度，如果你同我一样喜欢算法、LeetCode，可以关注我 GitHub 上的 LeetCode 题解：[Leetcode-Solutions-with-Java-And-Kotlin](https://github.com/hi-dhl/Leetcode-Solutions-with-Java-And-Kotlin)，一起来学习，期待与你一起成长。

### Android 10 源码系列

正在写一系列的 Android 10 源码分析的文章，了解系统源码，不仅有助于分析问题，在面试过程中，对我们也是非常有帮助的，如果你同我一样喜欢研究 Android 源码，可以关注我 GitHub 上的 [Android10-Source-Analysis](https://github.com/hi-dhl/Android10-Source-Analysis)，文章都会同步到这个仓库。

* [0xA01 Android 10 源码分析：APK 是如何生成的](https://juejin.im/post/5e4366c3f265da57397e1189)
* [0xA02 Android 10 源码分析：APK 的安装流程](https://juejin.im/post/5e5a1e6a6fb9a07cb427d8cd)
* [0xA03 Android 10 源码分析：APK 加载流程之资源加载](https://juejin.im/post/5e6c8c14f265da574b792a1a)
* [0xA04 Android 10 源码分析：APK 加载流程之资源加载（二）](https://juejin.im/post/5e7f0f2c51882573c4676bc7)
* [0xA05 Android 10 源码分析：Dialog 加载绘制流程以及在 Kotlin、DataBinding 中的使用](https://juejin.im/post/5e9199db6fb9a03c7916f635)
* [0xA06 Android 10 源码分析：WindowManager 视图绑定以及体系结构](https://juejin.im/post/5ead0b865188256d545fd2f8)
* [0xA07 Android 10 源码分析：Window 的类型 以及 三维视图层级分析](https://juejin.im/post/5ed98f6df265da770f52035c)
* [更多......](https://github.com/hi-dhl/Android10-Source-Analysis)

### 精选译文

目前正在整理和翻译一系列精选国外的技术文章，不仅仅是翻译，很多优秀的英文技术文章提供了很好思路和方法，每篇文章都会有**译者思考**部分，对原文的更加深入的解读，可以关注我 GitHub 上的 [Technical-Article-Translation](https://github.com/hi-dhl/Technical-Article-Translation)，文章都会同步到这个仓库。

* [[译][Google工程师] 刚刚发布了 Fragment 的新特性 “Fragment 间传递数据的新方式” 以及源码分析](https://juejin.im/post/5eb58da05188256d6d6bb248) 
* [[译][Google工程师] 详解 FragmentFactory 如何优雅使用 Koin 以及部分源码分析](https://juejin.im/post/5ecb16f1f265da76fb0c3967)
* [[译][2.4K Start] 放弃 Dagger 拥抱 Koin](https://juejin.im/post/5ebc1eb8e51d454dcf45744e?utm_source=gold_browser_extension)
* [[译][5k+] Kotlin 的性能优化那些事](https://juejin.im/post/5ec0f3afe51d454db11f8a94#heading-7)
* [[译] 解密 RxJava 的异常处理机制](https://juejin.im/post/5ecc10626fb9a047e25d5aac)
* [[译][1.4K+ Star] Kotlin 新秀 Coil VS Glide and Picasso](https://juejin.im/post/5edd1f5ae51d45789e0d9a22)
* [更多......](https://github.com/hi-dhl/Technical-Article-Translation)






