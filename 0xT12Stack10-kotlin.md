# Stack Overflow 上最热门的 10 个 Kotlin 问题？

![](https://img.hi-dhl.com/164907002588321.jpg)

> hi 大家好，我是 DHL。公众号：ByteCode ，专注分享最新技术原创文章，涉及 Kotlin、Jetpack、算法动画、数据结构 、系统源码、 LeetCode / 剑指 Offer / 多线程 / 国内外大厂算法题 等等。
> * 译文地址: [https://blog.autsoft.hu/top-10-kotlin......](https://blog.autsoft.hu/top-10-kotlin-stack-overflow-questions/)
> * 译者：DHL
> * 本文已收录于仓库 [Technical-Article-Translation](https://github.com/hi-dhl/Technical-Article-Translation)


这是 Stack Overflow 上最热门的几个 Kotlin 问题，每个问题如果更深入的分析，都可以单独写一篇文章，后面我会针对这些问题，在进一步的分析。

**通过这篇文章你将学习到以下内容：**

* `Array<Int>` 和 `IntArray` 的区别，以及如何选择
* `Iterable` 和 `Sequence` 的区别，以及如何选择
* 常用的 8 种 For 循环遍历的方法
* 在 Kotlin 中如何使用 SAM 转换
* 如何声明一个静态成员，Java 和 Koltin 进行互操作
* 为什么 kotlin 中的智能转换不能用于可变属性，如何才能解决这个问题
* 当重写 Java 函数时，如何决定参数的可空性
* 如何在一个文件中使用多个具有相同名称的扩展函数和类


## 译文

### Array<Int> 和 IntArray 的区别

**Array<Int>**

`Array<T>` 可以为任何 **T** 类型存储固定数量的元素。它和 `Int` 类型参数一起使用, 例如 `Array<Int>`，编译成 Java 代码，会生成  `Integer[]` 实例。我们可以通过 `arrayOf` 方法创建数组。

```
val arrayOfInts: Array<Int> = arrayOf(1, 2, 3, 4, 5)
```

**IntArray**

`IntArray` 可以让我们使用基础数据类型的数组，编译成 Java 代码，会生成 `int[]` (其它的基础类型的数组还有 `ByteArray` , `CharArray` 等等), 我们可以通过 `intArrayOf` 工厂方法创建数组。

```
val intArray: IntArray = intArrayOf(1, 2, 3, 4, 5)
```


**什么时候使用 `Array<Int>` 或者 IntArray**

默认使用 `IntArray`，因为它的性能更好，不需要对每个元素进行装箱。`IntArray` 进行初始化的时候，默认将每个索引的值初始化为 0，代码如下所示。

```
val intArray = IntArray(10)
val arrayOfInts = Array<Int>(5) { i -> i * 2 }
```

而 `Array<Int>` 的性能比较差，会对每个元素进行装箱，如果你需要创建包含 null 值的数组，Kotlin 也提供了 `arrayOfNulls` 方法，帮助我们进行创建。

```
val notActualPeople: Array<Person?> = arrayOfNulls<Person>(13)
```

### Iterable 和 Sequence 的区别

**Iterable**

`Iterable` 对应 Java 的 `java.lang.Iterable`, `Iterable` 会立即处理输入的元素，并返回一个包含结果的新集合。我们来举一个简单的例子 **返回年龄 > 21 前 5 个人的集合**。

```
val people: List<Person> = getPeople()
val allowedEntrance = people
		.filter { it.age >= 21 }
		.map { it.name }
		.take(5)
```

* 首先通过 `filter` 函数检查每个人的年龄，将结果放入到一个新的结果集中
* 通过 `map` 函数对上一步得到的结果进行名字映射，然后生成一个新的列表 `list<String>`
* 通过 `take` 函数返回前 5 个元素，得到最终的结果集

**Sequence**

`Sequence` 是 Kotlin 中一个新的概念，用来表示一个延迟计算的集合。`Sequence` 只存储操作过程，并不处理任何元素，直到遇到终端操作符才开始处理元素，我们也可以通过 `asSequence` 扩展函数，将现有的集合转换为 `Sequence` ，代码如下所示。

```
val people: List<Person> = getPeople()
val allowedEntrance = people.asSequence()
	.filter { it.age >= 21 }
	.map { it.name }
	.take(5)
	.toList()
```

在这个例子中， `toList()` 表示终端操作符，`filter` 、 `map` 、 `take` 都是中间操作符，返回 `Sequence` 实例，当 `Sequence` 遇到中间操作符时，只是存储操作过程，并不参与计算，直到遇到 `toList()`。

`Sequence` 的好处它不会生成中间结果集，直接对原始列表中的每一个人重复这个步骤，直到找到 5 个人，返回最终的结果集。

**应该如何选择**

如果数据量比较小，可以使用 Iterable。虽然会创建中间结果集，在数据不大的情况下，对性能的影响不会很严重。

如果处理的数据量比较大，Sequence 是最好的选择，因为不会创建中间结果集，内存开销更小。

### 常用的 8 种 For 循环遍历方法


我们经常会使用以下方法进行遍历。

```
for (i in 0..args.size - 1) {
	println(args[i])
}
```

但是 `Array` 有一个可读性更强的扩展属性 `lastIndex`。

```
for (i in 0..args.lastIndex) {
	println(args[i])
}
```

但是实际上我们不需要知道最后一个索引，有一个更加简单的写法。

```
for (i in 0 until args.size) {
	println(args[i])
}
```

当然你也可以使用下标扩展属性 `indices` 得到它的范围。

```
for (i in args.indices) {
	println(args[i])
}
```

还有一个更加直接的写法，通过下面的方式直接迭代集合。

```
for (arg in args) {
	println(arg)
}
```

您也可以使用 `forEach` 函数，传递一个 `lambda` 表达式来处理每个元素。

```
args.forEach { arg ->
	println(arg)
}
```

它们生成的 Java 代码都非常的相似，在这些例子中，都增加一个索引变量，并在循环中通过索引获取元素。但是如果我们迭代的是 `List`，最后两个例子底层使用 Iterator，而其他的例子仍是通过索引获取元素。另外还有两个遍历的方法：

* `withIndex` 函数，它返回一个 `Iterable` 对象，该对象可以被解构为当前索引和元素。

```
for ((index, arg) in args.withIndex()) {
    println("$index: $arg")
}
```

* `forEachIndexed` 函数，它为每个索引和参数提供了一个 lambda 表达式。

```
args.forEachIndexed { index, arg ->
    println("$index: $arg")
}
```

### 如何使用 SAM 转换

可以通过 lambda 表达式实现 SAM 转换，从而使代码更简洁，可读性更强，我们来看一个例子。

在 Java 中定义一个 `OnClickListener` 接口，并声明一个 `onClick` 的方法。


```
public interface OnClickListener {
    void onClick(Button button);
}
```

我们给 `Button` 添加 `OnClickListener` 监听器，每次点击的时候都会被调用。

```
public class Button {
    public void setListener(OnClickListener listener) { ... }
}
```

在 Kotlin 中常见的写法，创建匿名类，实现 `OnClickListener` 接口。

```
button.setListener(object: OnClickListener {
    override fun onClick(button: Button) {
        println("Clicked!")
    }
})
```

如果我们使用 SAM 转换，将使代码更简洁，可读性更强。

```
button.setListener {
    fun onClick(button: Button) {
        println("Clicked!")
    }
}
```

### 如何声明一个静态成员，Java 和 Koltin 进行互操作

一个普通的类可以有静态成员和非静态成员，在 Kotlin 中我们常把静态成员放到 `companion object` 中。

```
class Foo {
    companion object {
        fun x() { ... }
    }
    fun y() { ... }
}
```

我们也可以用 `object` 来声明一个单例，替换 `companion object`。

```
object Foo {
    fun x() { ... }
}
```

如果你不想总是通过类名去调用 `Foo.x()`, 而只是想使用 `x()`，可以声明为顶级函数。

```
fun x() { ... }
```

另外想在 Java 中调用 Kotlin 中静态方法，需要添加 `@JvmStatic` 和 `@JvmName` 注解。用一张表格汇总一下，如何在 Java 中调用 Kotlin 中的代码。


| Function declaration | Kotlin usage | Java usage |
| --- | --- | --- |
| Companion object | Foo. F () | Foo. Companion. F (); |
| Companion object with @JvmStatic | Foo. F () | Foo. F (); |
| Object | Foo. F () | Foo. INSTANCE. F (); |
| Object with @JvmStatic | Foo. F () | Foo. F (); |
| Top level function | f () | UtilKt. F (); |
| Top level function with @JvmName | f () | Util. F (); |


同样的规则也适用于变量，`@JvmField` 注解用于变量上，加上 `const` 关键字，编译时可以将常量值内联到调用处。

| Variable declaration | Kotlin usage | Java usage |
| --- | --- | --- |
| Companion object | X. X | X. Companion. GetX (); |
| Companion object with @JvmStatic | X. X | X. GetX (); |
| Companion object with @JvmField | X. X | X. X; |
| Companion object with const | X. X | X. X; |
| Object | X. X | 	X. INSTANCE. GetX (); |
| Object with @JvmStatic | X. X | X. GetX (); |
| Object with @JvmField | X. X | X. X |
| Object with const | X. X | X. X; |
| Top level variable | X. X | ConstKt. GetX (); |
| Top level variable with @JvmField | X. X | ConstKt. X; |
| Top level variable with const | x| ConstKt. X; |
| Top level variable with @JvmName | x | Const. GetX (); |
| Top level variable with @JvmName and @JvmField | x | Const. X; |
| Top level variable with @JvmName and const | x | Const. X; |


### 为什么 kotlin 中的智能转换不能用于可变属性


我们先来看一段有问题的代码。

```
class Dog(var toy: Toy? = null) {
    fun play() {
        if (toy != null) {
            toy.chew()
        }
    }
}
```

上面的代码在编译时无法通过，异常信息如下所示。

```
Kotlin: Smart cast to 'Toy' is impossible, because 'toy' is a mutable property that could have been changed by this time
```

出现这个问题的原因在于，执行完 `toy != null` 之后和 `toy.chew()` 方法被调用之间，这个 `Dog` 的实例可能被另外一个线程修改，这可能会出现 `NullPointerException` 异常。

**如何才能解决这个问题呢**

只需要将变量设置为不可变的，即用 `val` 声明，那么上面的问题就不存在，默认情况将所有的变量都用 `val` 声明，除非有必要的时候，才将它们设置为 `var`。


如果一定要声明为 `var` ，那么可以使用局部不可变的副本来解决这个问题。修改一下上面的代码，如下所示。

```
class Dog(var toy: Toy? = null) {
    fun play() {
        val _toy = toy
        if (_toy != null) {
            _toy.chew()
        }
    }
}
```

但是还有一个更简洁的写法。

```
class Dog(var toy: Toy? = null) {
    fun play() {
        toy?.length
    }
}
```

### 当重写 Java 函数时，如何决定参数的可空性


在 Java 中定义一个 `OnClickListener` 接口，并声明一个 `onClick` 的方法。


```
public interface OnClickListener {
    void onClick(Button button);
}
```


在 Kotlin 中实现这个接口，并通过 `IDEA` 自动生成 `onClick` 方法，将会得到下面的方法签名，`onClick` 方法参数默认为可空类型。

```
class KtListener: OnClickListener {
    override fun onClick(button: Button?): Unit {
        val name = button?.name ?: "Unknown button"
        println("Clicked $name")
    }
}
```

由于 Java 平台没有可空类型，而 Kotlin 中有，在这个例子中 Button 是否为空由我们来决定。默认情况下，对所有参数使用可空类型更安全，编译器会强制我们处理这些参数。

对于已知的永远不会空的参数，可以使用非空类型，空和非空都可以正常编译，但是如果将方法参数声明为非空，那么 Kotlin 编译器会自动注入一个空的检查，可能会抛出  `IllegalArgumentException` 异常，潜在的风险很大。当然使用非空参数，代码将会更加简洁。

```
class KtListener: OnClickListener {
    override fun onClick(button: Button): Unit {
        val name = button.name
        println("Clicked $name")
    }
}
```

### 如何在一个文件中使用多个具有相同名称的扩展函数和类

假设在不同的包中对 String 类实现了两个相同名字的扩展函数，如果是一个普通函数，你可以使用完全限定包名来调用它，但是扩展函数不行。所以我们可以在 `import` 语句中使用 `as` 关键字对其重命名，代码如下所示。

```
import com.example.code.indent as indent4
import com.example.square.indent as indent2

"hello world".indent4()
```

另外一个案例，想在同一个文件中使用来自不同包中两个具有相同名称的类（例如 `java.util.Date` 和 `java.sql.Date` ），并且您不希望通过完全限定包名来调用它们。我们也可以在 `import` 语句中使用 `as` 关键字对其重命名。

```
import java.util.Date as UtilDate
import java.sql.Date as SqlDate
```

现在我们就可以在这个类中，使用通过  `as` 关键字声明的别名来引用这些类。



## 译者

全文到这里就结束了，这篇文章每个问题，都是一个知识点，后面我会针对每个问题，单独写一篇文章，进行更加深入的分析。

<p style="text-align: center; margin-top: 10px;">
    <b>如果有帮助点个赞就是对我最大的鼓励</b>
</p>
<p style="text-align: center; margin-top: 10px;">
    <b>代码不止，文章不停</b>
</p>
<p style="text-align: center; margin-top: 10px;">
    <b>欢迎关注公众号：ByteCode，持续分享最新的技术</b>
</p>

---

最后推荐长期更新和维护的项目：

* 个人博客，将所有文章进行分类，欢迎前去查看 [https://hi-dhl.com](https://hi-dhl.com)

* KtKit 小巧而实用，用 Kotlin 语言编写的工具库，欢迎前去查看 [KtKit](https://github.com/hi-dhl/KtKit)

* 计划建立一个最全、最新的 AndroidX Jetpack 相关组件的实战项目以及相关组件原理分析文章，正在逐渐增加 Jetpack 新成员，仓库持续更新，欢迎前去查看 [AndroidX-Jetpack-Practice](https://github.com/hi-dhl/AndroidX-Jetpack-Practice)

* LeetCode / 剑指 offer / 国内外大厂面试题 / 多线程题解，语言 Java 和 kotlin，包含多种解法、解题思路、时间复杂度、空间复杂度分析<br/>

![](https://img.hi-dhl.com/cde6d3ba158742d1a821390fad1111.png)


* 剑指 offer 及国内外大厂面试题解：[在线阅读](https://offer.hi-dhl.com)
* LeetCode 系列题解：[在线阅读](https://leetcode.hi-dhl.com)



**近期必读热门文章**

* [value class 完全代替 typealias？](https://mp.weixin.qq.com/s/qbsslnyp-WGBRubFdVahDA)
* [容易被忽视的几个 Kotlin 细节, value class 执行效率竟然这么高](https://mp.weixin.qq.com/s/-2_fJ7vLxXb8_DAcvescOw)
* [Kotlin 宣布一个重磅特性](https://mp.weixin.qq.com/s/oS8r2DieTYBhFuyWF6jx9w)
* [1分钟发布一个网站，不用域名、服务器，网站发布从未如此简单](https://mp.weixin.qq.com/s/JvQ3zSbyobVqgUSZAoZq3Q)
* [这是最棒的效率工具集，打通 Notion x 云盘 x 其他笔记软件，写作、设计、开发都会用到工具](https://mp.weixin.qq.com/s/mshdTnxML-BtfcgO15ni5Q)
* [Android 12 已来，你的 App 崩溃了吗？](https://mp.weixin.qq.com/s/NuqAYoUq_0OorM1rVHUEHA)
* [Google 宣布废弃 LiveData.observe 方法](https://mp.weixin.qq.com/s/fp1ZOmqAcEBv2f7ec1r-zw)
* [使用 kotlin 需要注意的一个细节](https://mp.weixin.qq.com/s/7ZoBeSK97j4YWXTGX9vEYA)
* [影响性能的 Kotlin 代码（一）](https://mp.weixin.qq.com/s/8dAbt1-mcCVLWLXKC-1_xw)
* [Jetpack Splashscreen 解析 | 助力新生代 IT 农民工 事半功倍](https://mp.weixin.qq.com/s/1jTdPgXbNl38smOnN2k1MA)
* [为数不多的人知道的 Kotlin 技巧及解析(三)](https://mp.weixin.qq.com/s/lcLJB0MFaYX1lQXtJ3M88g)
* [揭秘 Kotlin 中的 == 和 ===](https://mp.weixin.qq.com/s/sYj_-wqENr9Jaw1p8iP4Jg)
* [Kotlin 密封类进化了](https://mp.weixin.qq.com/s/0O-ZnbVnHUnh3jCnthKgHA)





