# 避免滥用 Kotlin 扩展函数

![](http://img.hi-dhl.com/kotlin_extend_fun_1.png)


> hi 大家好，我是 DHL。公众号：ByteCode ，专注分享最新技术原创文章，涉及 Kotlin、Jetpack、算法动画、数据结构 、系统源码、 LeetCode / 剑指 Offer / 多线程 / 国内外大厂算法题 等等。
> 
> * 原标题：Effective Kotlin Item 46: Avoid member extensions
> * 原地址：[kt.academy/article/ek-member-extensions......](https://kt.academy/article/ek-member-extensions?utm_campaign=onCreate%20Digest&utm_medium=email&utm_source=Revue%20newsletter)
> * 原作者：Marcin Moskała
> * 译者：DHL
> * 本文已收录于仓库 [Technical-Article-Translation](https://github.com/hi-dhl/Technical-Article-Translation)


当我们为类定义扩展函数时，它不会作为成员添加到类中。扩展函数是一种特殊的函数，它默认的第一个参数是函数的接受者，如下例所示，扩展函数被编译成普通函数。

```
fun String.isPhoneNumber(): Boolean =
    length == 7 && all { it.isDigit() }
```

编译成一个类似于扩展函数的函数

```
fun isPhoneNumber(`$this$: String): Boolean =
    $this$.length == 7 && $this`.all { it.isDigit() }
```

除了为类定义扩展函数之外，还可以定义成员扩展，甚至还可以在接口中定义扩展。

```
interface PhoneBook {
    fun String.isPhoneNumber(): Boolean
}

class Fizz : PhoneBook {
    override fun String.isPhoneNumber(): Boolean =
        this.length == 7 && this.all { it.isDigit() }
}
```

不要仅仅为了限制可见性，而将函数定义为成员扩展函数，如下所示。

```
// Bad practice, do not do this
class PhoneBookIncorrect {

    fun verify(number: String): Boolean {
        require(number.isPhoneNumber())
        // ...
    }

    // ...

    fun String.isPhoneNumber(): Boolean =
        this.length == 7 && this.all { it.isDigit() }
}
```

其实这样做并没有真正限制可见性，它只会使扩展函数变得更加复杂，调用的时候，需要同时提供扩展接受者和调度接受者。


```
PhoneBookIncorrect().apply {
    "1234567890".isPhoneNumber()
}
```

你应该使用可见修饰符，限制扩展函数的可见性，而不是将其设置成员扩展函数。

```
class PhoneBook {

    fun verify(number: String): Boolean {
        require(number.isPhoneNumber())
        // ...
    }

    // ...
}

// This is how we limit extension functions visibility
private fun String.isPhoneNumber(): Boolean =
    this.length == 7 && this.all { it.isDigit() }
```

如果你需要将一个函数作为成员，并且希望像调用扩展函数一样使用它，请考虑使用let。

```
class PhoneBook(
    private val phoneNumberVerifier: PhoneNumberVerifier
) {

    fun verify(number: String): Boolean {
        require(number.let(::isPhoneNumber))
    }

    private fun isPhoneNumber(number: String): Boolean =
        phoneNumberVerifier.verify(number)
}
```

**为什么需要避免成员扩展函数**

建议尽量避免使用成员扩展函数，主要有以下几个原因：

* 不支持引用

```
val ref = String::isPhoneNumber
val str = "1234567890"
val boundedRef = str::isPhoneNumber

val refX = PhoneBookIncorrect::isPhoneNumber // ERROR
val book = PhoneBookIncorrect()
val boundedRefX = book::isPhoneNumber // ERROR
```

* 两个接受者隐式的访问可能会令人困惑

```
class A {
    val a = 10
}
class B {
    val a = 20
    val b = 30

    fun A.test() = a + b // Is it 40 or 50?
}
```


* 当我们期望修改引用接受者的时候，我们不清楚是修改的是扩展接受者还是调度接受者

```
class A {
    //...
}
class B {
    //...

    fun A.update() ... // Does it update A or B?
}
```

* 对于经验较少的开发人员来说，看到成员扩展可能是违反直觉，可读性很差。

**避免,而不是禁止**

这条规则并不适用于任何地方，最明显的情况是，当我们定义 DSL 时，需要使用成员扩展。

> DSL 全称 Domain Specific Language 即 领域特定语言，在 Kotlin 中最主要的实现方式是高阶函数

当需要调用在某个作用域上定义的函数时，成员扩展也是非常有用的，举两个例子，一个例子可能是使用 produce 生成 Channel 的成员函数，另一个是定义在接口中的，集成测试函数。

```
class OrderUseCase(
    // ...
) {
    // ...

    private fun CoroutineScope.produceOrders() =
        produce<Order> {
            var page = 0
            do {
                val orders = api
                    .requestOrders(page = page++)
                    .orEmpty()
                for (order in orders) send(order)
            } while (orders.isNotEmpty())
        }
}

interface UserApiTrait {

    fun TestApplicationEngine.requestRegisterUser(
        token: String,
        request: RegisterUserRequest
    ): UserJson? = ...

    fun TestApplicationEngine.requestGetUserSelf(
        token: String
    ): UserJson? = ...

    // ...
}
```

我们建议尽可能避免定义成员扩展函数，但是如果它们是最好的选择，我们还是会使用它们。

**总结**

本篇文章主要介绍了，应当尽量避免使用成员扩展函数，除了特殊的场景例如 DSL， 因为成员扩展函数存在很多缺点，我们应该尽量避免，这只是建议，不是强制，更不应该使用成员扩展函数来限制可见性，你应该使用可见修饰符，限制扩展函数的可见性。

<p align="center">
<br/>
<p align="center"><b>如果有帮助 点个赞 就是对我最大的鼓励</b></p>
<p align="center"><b>代码不止，文章不停</b></p>
<p align="center"><b>欢迎关注公众号：ByteCode，持续分享最新的技术</b></p>
<br/>
</p>

---

最后推荐长期更新和维护的项目：

* 个人博客，将所有文章进行分类，欢迎前去查看 [https://hi-dhl.com](https://hi-dhl.com)

* KtKit 小巧而实用，用 Kotlin 语言编写的工具库，欢迎前去查看 [KtKit](https://github.com/hi-dhl/KtKit)

* 计划建立一个最全、最新的 AndroidX Jetpack 相关组件的实战项目 以及 相关组件原理分析文章，正在逐渐增加 Jetpack 新成员，仓库持续更新，欢迎前去查看 [AndroidX-Jetpack-Practice](https://github.com/hi-dhl/AndroidX-Jetpack-Practice)

* LeetCode / 剑指 offer / 国内外大厂面试题 / 多线程 题解，语言 Java 和 kotlin，包含多种解法、解题思路、时间复杂度、空间复杂度分析<br/>

    <image src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cde6d3ba158742d1a821390fad86d50c~tplv-k3u1fbpfcp-zoom-1.image" width = "500px"/>
  
    * 剑指 offer 及国内外大厂面试题解：[在线阅读](https://offer.hi-dhl.com)
    * LeetCode 系列题解：[在线阅读](https://leetcode.hi-dhl.com)

**近期必读热门文章**

* [Android 12 已来，你的 App 崩溃了吗？](https://mp.weixin.qq.com/s/NuqAYoUq_0OorM1rVHUEHA)
* [Android 进化史 1.0 到 12 ，还记得第一次使用是哪个版本？](https://mp.weixin.qq.com/s/nQO6ba7yJ6e_cBhmdn8M8g)
* [LinkedList 落幕了吗？](https://mp.weixin.qq.com/s/YK0yG1jt9W9Zbb2wbXIE5g)
* [Oracle 官方推荐，使用 ReentrantLock 需要注意的细节](https://mp.weixin.qq.com/s/9NMpEP-7mABh44Wc3-E8DA)
* [Kotlin 宣布一个重磅特性](https://mp.weixin.qq.com/s/oS8r2DieTYBhFuyWF6jx9w)
* [Google 宣布废弃 LiveData.observe 方法](https://mp.weixin.qq.com/s/fp1ZOmqAcEBv2f7ec1r-zw)
* [使用 kotlin 需要注意的一个细节](https://mp.weixin.qq.com/s/7ZoBeSK97j4YWXTGX9vEYA)
* [影响性能的 Kotlin 代码（一）](https://mp.weixin.qq.com/s/8dAbt1-mcCVLWLXKC-1_xw)
* [Jetpack Splashscreen 解析 | 助力新生代 IT 农民工 事半功倍](https://mp.weixin.qq.com/s/1jTdPgXbNl38smOnN2k1MA)
* [为数不多的人知道的 Kotlin 技巧及解析(三)](https://mp.weixin.qq.com/s/lcLJB0MFaYX1lQXtJ3M88g)
* [为数不多的人知道的 Kotlin 技巧以及 原理解析（二）](https://juejin.im/post/6847902224467623950)
* [为数不多的人知道的 Kotlin 技巧以及 原理解析（一）](https://juejin.im/post/5edfd7c9e51d45789a7f206d)
* [揭秘 Kotlin 中的 == 和 ===](https://mp.weixin.qq.com/s/sYj_-wqENr9Jaw1p8iP4Jg)
* [Kotlin 密封类进化了](https://mp.weixin.qq.com/s/0O-ZnbVnHUnh3jCnthKgHA)
* [Kotlin 中的密封类 优于 带标签的类](https://mp.weixin.qq.com/s/VNxFVGvDU-X3lfenehkGtw)
* [Kotlin Sealed 是什么？为什么 Google 都在用](https://mp.weixin.qq.com/s/X1jEUFS8vmRF9ybkmVy5zQ)





