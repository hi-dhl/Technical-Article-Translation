# 0xT04 [译][5k+] Kotlin 的性能优化那些事

 ![](http://cdn.51git.cn/2020-06-13-15897114925544.jpg)

## 前言

> * 原标题:  Item: Consider aggregating elements to a map
> * 原文地址: [https://blog.kotlin-academy.com/item......](https://blog.kotlin-academy.com/item-consider-aggregating-elements-to-map-a5a35a7b6c61)
> * 原文作者：Marcin Moskala
> * 介绍：作者 Marcin Moskala 是大神级别的人物，在 Medium 上至少有 5K+ 的关注者，在 Twitter 上至少有 4K+ 的关注者，是 「Effective Kotlin」一书的作者之一。「Effective Kotlin」总结了 Kotlin 社区的最佳实践和经验，很多来自 Google 工程师的案例，揭露了很多不为人知的 Kotlin 背后的魔法。

这篇文章应该可以说是 [[译][2.4K Start] 放弃 Dagger 拥抱 Koin](https://juejin.im/post/5ebc1eb8e51d454dcf45744e?utm_source=gold_browser_extension) 文章的续集，在 “放弃 Dagger 拥抱 Koin” 文章中介绍了过渡使用 Inline 修饰符所带来的后果，以及 Koin 团队在为修复 1x 版本所做的性能优化，这边文章将继续学习如何提升 Kotlin 的查询速度。

**通过这篇文章你将学习到以下内容，将在译者思考部分会给出相应的答案**

* 如何提升 Kotlin 的查询速度？
* 性能和代码可读性该做如何选择？
* Kotlin 内存泄露那些事, 消除过期的对象引用？
* 如何提高 Kotlin 代码的可读性？
* Kotlin 算法：一行代码实现杨辉三角？

这篇文章涉及很多重要的知识点，带着自己理解，请耐心读下去，应该可以从中学到很多技巧

## 译文

我们需要多次访问大量的数据情况，这其实并不少见，例如：

* cache：从服务上下载的数据，然后保存在本地内存中以更快地访问它们
* repository：从一些文件中加载数据
* in-memory repository：用于不同类型的内存测试

这些数据可能表示一些用户、id、配置等等，它们通常以 list 形式返给我们，它们可能以相同的方式存储在内存中：

```
class NetworkUserRepo(val userService: UserService): UserRepo {
    private var users: List<User>? = null
    override fun getUser(id: UserId): User? {
        if(users == null) {
            users = userService.getUsers()
        }
        return users?.firstOrNull { it.id == id }
    }
}

class ConfigurationsRepository(
    val configurations: List<Configuration>
) {
    fun getByName(name: String) = configurations
        .firstOrNull { it.name == name }
}
class InMemoryUserRepo: UserRepo {
   private val users: MutableList<User> = mutableListOf()
   override fun getUser(id: UserId): User?
      = users.firstOrNull { it.id == id }
   
   fun addUser(user: User) {
      user.add(user)
   }
}
```

这可能是存储这些元素的最好方式，注意我们是如何加载数据如何使用的，我们通过某个标识符或者名字访问这些元素（它们与我们设计数据库时唯一值有关），当 n 等于 list 的大小时，在 list 中查找元素的复杂度为 O(n)，更准确的说，平均需要 n / 2 次比较才能找到一个元素，如果是一个比较的大的 list，查找效率极其低效，解决这个问题的一个好办法是使用 Map 代替 list, Kotlin 默认使用的是 hash map, 更具体的说是 LinkedHashMap，当我们使用 hash map 查找元素的性能要好得多, 实际上 JVM 使用的 hash map 的大小根据映射本身的大小进行了调整, 如果实现 hashCode 方式正确，查找一个元素只需要进行一次比较。

这是 InMemoryRepo 中使用 map 代替 list

```
class InMemoryUserRepo: UserRepo {
   private val users: MutableMap<UserId, User> = mutableMapOf()
   override fun getUser(id: UserId): User? = users[id]
   
   fun addUser(user: User) {
      user.put(user.id, user)
   }
}
```

大多是其他操作，比如修改或者迭代这些数据（可能使用集合方法 filter, map, flatMap, sorted, sum 等等）对于 list 和 map 性能差不多的。

那么我们如何从 list 转换到 map，或者从 map 转换到 list，使用 associate 方法来完成 list 转换到 map，最常见的方法是 associateBy，它构建一个映射，其中的值是列表中的元素，键是通过一个 lambda 表达式提供。

```
data class User(val id: Int, val name: String)
val users = listOf(User(1, "Michal"), User(2, "Marek"))
val byId = users.associateBy { it.id }
byId == mapOf(1 to User(1, "Michal"), 2 to User(2, "Marek")) 
val byName = users.associateBy { it.name }
byName == mapOf("Michal" to User(1, "Michal"), 
              "Marek" to User(2, "Marek"))
```

注意，映射中的键必须是唯一的，否则相同键值的元素会被删掉，这就是为什么我们应该根据唯一标识符进行关联（对于键值不是唯一的，应该使用 groupBy 方法）

```
val users = listOf(User(1, "Michal"), User(2, "Michal"))
val byName = users.associateBy { it.name }
byName == mapOf("Michal" to User(2, "Michal"))
```

从 map 转换到 list 使用 values 方法

```
val users = listOf(User(1, "Michal"), User(2, "Michal"))
val byId = users.associateBy { it.id }
users == byId.values
```

如何在 repositories 中用 Map 提高元素访问的性能

```
class NetworkUserRepo(val userService: UserService): UserRepo {
    private var users: Map<UserId, User>? = null
    override fun getUser(id: UserId): User? {
        if(users == null) {
            users = userService.getUsers().associateBy { it.id }
        }
        return users?.get(id)
    }
}

class ConfigurationsRepository(
    configurations: List<Configuration>
) {
    val configurations: Map<String, Configuration> = 
        configurations.associateBy { it.name }
    
    fun getByName(name: String) = configurations[name]
}
```

这个技巧是非常重要的，但是并不适合所有的 cases，当我们需要访问比较大的 list 的时候是非常有用的，这在后台访问是非常重要的，这些 list 可能在后台每秒被访问很多次，但是在前台并不重要（这里说的是 Android 或者 iOS）用户最多只会访问几次  repository，需要注意的是从 list 转换到 map 是需要时间的，如果过渡使用，可能会对性能有不好的影响。

## 译者思考

作者总共从三个方面 Network、Configurations、InMemory 告诉我们应该如何从 list 转 map, 或者从 map 转 list, 以及应该在后台需要多次访问很大的数据集合中使用 map，过渡的使用只会对性能产生负面的影响。

* list 转 map 调用用 associateBy 方法，接受一个 lambda 表达式

```
val users = listOf(User(1, "Michal"), User(2, "Michal"))
val byName = users.associateBy { it.name }
byName == mapOf("Michal" to User(2, "Michal"))
```

* 从 map 转 list 调用 values 方法

```
val users = listOf(User(1, "Michal"), User(2, "Michal"))
val byId = users.associateBy { it.id }
users == byId.values
```

这是一个非常重要的优化的手段（使用空间换取时间），在 [[译][2.4K Start] 放弃 Dagger 拥抱 Koin](https://juejin.im/post/5ebc1eb8e51d454dcf45744e?utm_source=gold_browser_extension) 文章中介绍了当我们引入 Koin 1x 的时候冷启动时间变长了，而且在有大量依赖的时候，查找的时间会有点长，用过这个版本的朋友，应该都会有这个感觉，Koin 团队的解决方案中用到了 HashMap，使用空间换取时间，查找一个 Definition 时间复杂度变成了 O(1)，从提高的访问速度。

**其实我们应该在头脑中，保持内存管理的意识，在每次优化、修改代码之前，不要急于写代码，先整理一下思路，在头脑中过一遍自己的方案，我们应该为项目找到一个折衷方案，不仅要考虑内存和性能，还要考虑代码的可读性。当我们做一个应用程序，在大多数情况下可读性更重要。当我们开发一个库时，通常性能和内存更重要。**

### 性能和代码可读性该做如何选择

如果用 Java 和 Kotlin 语言刷过 LeetCode，使用相同的思路实现同一个算法，在正常的 Case 中，Kotlin 和 Java 执行时间差值很小，数据量越大的情况下 Kotlin 和 Java 差距会越来越大，Kotlin 执行时间会越来越慢，但是为什么 Kotlin 语言还会成为 Android 开发的首选语言呢？来看一下作者 Marcin Moskala 另外一篇文章 [My favorite examples of functional programming in Kotlin](https://www.freecodecamp.org/news/my-favorite-examples-of-functional-programming-in-kotlin-e69217b39112/) 展示的快排算法。

在之前的文章中分享了过这个算法，现在我们来分析一下这个算法。

```
fun <T : Comparable<T>> List<T>.quickSort(): List<T> = 
    if(size < 2) this
    else {
        val pivot = first()
        val (smaller, greater) = drop(1).partition { it <= pivot}
        smaller.quickSort() + pivot + greater.quickSort()
    }
    
// 使用 [2,5,1] -> [1,2,5]
listOf(2,5,1).quickSort() // [1,2,5]
```

这是一个非常酷的函数式编程的例子，当看到这个算法的第一感觉，它非常的简洁，可读性很强，其次我们来看一下这个算法执行时间，其实它根本没有针对性能进行优化。

如果你需要使用高性能的算法，你可以使用 Java 标准库当中的函数，Kotlin 扩展函数 sorted() 就是用 Java 标准库中的函数，Java 标准库中的函数效率会更高的，但是实际执行时间怎么样呢？生成一个随机数数组，使用使用 quickSort() 和 sorted() 方法进行排序，比较它们的执行时间，代码如下所示：

```
val r = Random()
listOf(100_000, 1_000_000, 10_000_000)
    .asSequence()
    .map { (1..it).map { r.nextInt(1000000000) } }
    .forEach { list: List<Int> ->
        println("Java stdlib sorting of ${list.size} elements took ${measureTimeMillis { list.sorted() }}")
        println("quickSort sorting of ${list.size} elements took ${measureTimeMillis { list.quickSort() }}")
    }
```

执行结果如下所示：

```
Java stdlib sorting of 100000 elements took 83
quickSort sorting of 100000 elements took 163
Java stdlib sorting of 1000000 elements took 558
quickSort sorting of 1000000 elements took 859
Java stdlib sorting of 10000000 elements took 6182
quickSort sorting of 10000000 elements took 12133`
```

正如你所见，quickSort() 比 sorted() 排序算法要慢两倍，在正常情况下，差值通常在 0.1ms 和 0.2ms 之间，基本上可以忽略不计，但是它更简洁，可读性更强。这解释了在某些情况下，我们可以考虑使用一个优化程度稍低，但可读性强且简洁的函数，你同意作者这种观点吗？

### Kotlin 内存泄露那些事, 消除过期的对象引用

我看过很多文章都说 Kotlin 简洁和高效，Kotlin 确实很简洁，在 “如何提高 Kotlin 代码的可读性” 部分我会列举一些例子，但是高效的背后是有代价的，这块往往很容易被我们忽略，这就需要我们去研究 kotlin 语法糖背后的魔法，当我们在开发的时候，选择合适的语法糖，尽量避免这些错误，例如带有 lnmba 表达式高阶函数，不使用 Inline 修饰符，会被编译成匿名内部类等等，更详细的内容参考 [[译][2.4K Start] 放弃 Dagger 拥抱 Koin](https://juejin.im/post/5ebc1eb8e51d454dcf45744e?utm_source=gold_browser_extension) Inline 修饰符带来的性能损失部分。

**内存管理最重要的一条规则是，不使用的对象应该被释放**

这篇文章 [Effective Java in Kotlin, item 7: Eliminate obsolete object references](https://blog.kotlin-academy.com/effective-java-in-kotlin-item-7-eliminate-obsolete-object-references-9a197b6fb728) 作者也列举了 Kotlin 的一些例子，例如我们需要使用 mutableLazy 属性委托，像 lazy 一样工作，我们来看一下实现代码：

```
fun <T> mutableLazy(initializer: () -> T): ReadWriteProperty<Any?, T> = MutableLazy(initializer)

private class MutableLazy<T>(
    val initializer: () -> T
) : ReadWriteProperty<Any?, T> {

    private var value: T? = null
    private var initialized = false

    override fun getValue(
        thisRef: Any?, 
        property: KProperty<*>
    ): T {
        synchronized(this) {
            if (!initialized) {
                value = initializer()
                initialized = true
            }
            return value as T
        }
    }

    override fun setValue(
        thisRef: Any?, 
        property: KProperty<*>, 
        value: T
    ) {
        synchronized(this) {
            this.value = value
            initialized = true
        }
    }
}
```

如何使用：

```
var game: Game? by mutableLazy { readGameFromSave() }

fun setUpActions() {
    startNewGameButton.setOnClickListener {
        game = makeNewGame()
        startGame()
    }
    resumeGameButton.setOnClickListener {
        startGame()
    }
}
```

思考一下 mutableLazy 实现正确吗？ 它有一个地方不对，lnmba 表达式 initializer 在使用后没有被删除。这意味着只要对 MutableLazy 实例的引用存在，它就会被保持，即使它不再有用，如何改进 MutableLazy 实现的方法，优化代码如下所示：

```
fun <T> mutableLazy(initializer: () -> T): ReadWriteProperty<Any?, T> = MutableLazy(initializer)

private class MutableLazy<T>(
    var initializer: (() -> T)?
) : ReadWriteProperty<Any?, T> {

    private var value: T? = null

    override fun getValue(
        thisRef: Any?, 
        property: KProperty<*>
    ): T {
        synchronized(this) {
            val initializer = initializer
            if (initializer != null) {
                value = initializer()
                this.initializer = null
            }
            return value as T
        }
    }

    override fun setValue(
        thisRef: Any?, 
        property: KProperty<*>, 
        value: T
    ) {
        synchronized(this) {
            this.value = value
            this.initializer = null
        }
    }
}
```

在使用完之后将 initializer 设置为 null，它将会被 GC 回收。特别要注意当一个高阶函数会被编译成匿名类时或者它是一个未知类（任何或泛型类型）时，这个优化显得非常重要，我们来看一下 Kotlin stdlib 库中的类 SynchronizedLazyImpl 代码如下所示：
**kotlin-stdlib....../kotlin/util/LazyJVM.kt**

```
private class SynchronizedLazyImpl<out T>(
    initializer: () -> T, lock: Any? = null
) : Lazy<T>, Serializable {
    private var initializer: (() -> T)? = initializer
    private var _value: Any? = UNINITIALIZED_VALUE
    private val lock = lock ?: this

    override val value: T
        get() {
            val _v1 = _value
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }

            return synchronized(lock) {
                val _v2 = _value
                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST") (_v2 as T)
                } else {
                    val typedValue = initializer!!()
                    _value = typedValue
                    initializer = null
                    typedValue
                }
            }
        }
    ......
}
```

请注意，在使用完之后 initializers 设置为 null，将会被 GC 回收

### 如何提高 Kotlin 代码的可读性

上文提到了 Kotlin 简洁可读性很强，但是呢通过 AndroidStudio 提供了 convert our Java code to Kotlin 插件，将 Java 代码转换为 Kotlin 代码，Java-Style Kotlin 的代码明显很难看，那么如何提升 Kotlin 代码的可读性，我想分享几个很酷的例子 [Improve Java to Kotlin code review](https://medium.com/@elye.project/improve-java-to-kotlin-code-review-61e7bbb2c663)，用到了 Elvis 表达式、run, with 等等函数

**消除!!**

``` 
myList!!.length 
```

change to

``` 
myList?.length 
```

**空检查**

```
if (callback != null) {              
    callback!!.response()
}
```

change to

```
callback?.response()
```

**使用 Elvis 表达式**

```
if (toolbar != null) {
  if (arguments != null) {                  
    toolbar!!.title = arguments!!.getString(TITLE)              
  } else {                
    toolbar!!.title = ""            
  }
}
```

change to

```
toolbar?.title = arguments?.getString(TITLE) ?: “”
```

**使用 scope 函数**

```
val intent = intentUtil.createIntent(activity!!.applicationContext) 
activity!!.startActivity(intent)
dismiss()
```

change to

```
activity?.run { 
    val intent = intentUtil.createIntent(this)        
    startActivity(intent) 
    dismiss() 
}
```

ps: scope 函数还有 run, with, let, also and apply，它们的区别是什么，如何正确使用它们，后面的文章会详细的介绍。

**使用 takeIf if 函数**

```
if (something != null && something == preference) {   
     something.doThing() 
```

change to

```
something?.takeIf { it == preference }?.let { something.doThing() }
```

**Android TextUtil**

```
if (TextUtils.isEmpty(someString)) {...}
val joinedString = TextUtils.join(COMMA, separateList)
```

change to

```
if (someString.isEmpty()) {...}
val joinedString = separateList.joinToString(separator = COMMA)
```

**Java Util**

```
val strList = Arrays.asList("someString")
```

change to

```
val strList = listOf("someString")
```

**Empty and null**

```
if (myList == null || myList.isEmpty()) {...}
```

change to

```
if (myList.isNullOrEmpty() {...}
```

**避免对对象进行重复操作**

```
recyclerView.setLayoutManager(layoutManager)
recyclerView.setAdapter(adapter) 
recyclerView.setItemAnimator(animator)
```

change to

```
with(recyclerView) {
    setLayoutManager(layoutManager)         
    setAdapter(adapter)         
    setItemAnimator(animator)
}
```

**避免列表循环**

```
for (str in stringList) {
    println(str)
}
```

change to

```
stringList.forEach { println(it) }
```

**避免使用 mutable 集合**

```
val stringList: List<String> = mutableListOf()
for (other in otherList) {
    stringList.add(dosSomething(other))
}
```

change to

```
val stringList = otherList.map { dosSomething(it) }
```

**使用 when 代替 if**

```
if (requestCode == REQUEST_1) {            
    doThis()
} else if (requestCode == REQUEST_2) {
    doThat()
} else {
    doSomething()
}
```

change to

```
when (requestCode) { 
    REQUEST_1 -> doThis()
    REQUEST_1 -> doThat()
    else -> doSomething()
}
```

**使用 const**

```
companion object {        
    val EXTRA_STRING = "EXTRA_EMAIL"
    val EXTRA_NUMBER = 12345
}
```

change to

```
companion object {        
    const val EXTRA_STRING = "EXTRA_EMAIL"
    const val EXTRA_NUMBER = 12345
}
```

如果有更好的例子，欢迎留言

### Kotlin 算法：一行代码实现杨辉三角

我想分享一个很酷的算法，用一行代码实现杨辉三角，代码来自 Marcin Moskala 大神的 [Twitter](https://twitter.com/marcinmoskala/status/1018786619636224000?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed&ref_url=https%3A%2F%2Fcdn.embedly.com%2Fwidgets%2Fmedia.html%3Ftype%3Dtext%252Fhtml%26key%3Da19fcc184b9711e1b4764040d3dc5c07%26schema%3Dtwitter%26url%3Dhttps%253A%2F%2Ftwitter.com%2Fmarcinmoskala%2Fstatus%2F1018786619636224000%26image%3Dhttps%253A%2F%2Fi.embed.ly%2F1%2Fimage%253Furl%253Dhttps%25253A%25252F%25252Fpbs.twimg.com%25252Fmedia%25252FDiN0gXpXcAAzqdR.jpg%25253Alarge%2526key%253Da19fcc184b9711e1b4764040d3dc5c07)

```
fun pascal() = generateSequence(listOf(1)) { prev ->
    listOf(1) + (1..prev.lastIndex).map { prev[it - 1] + prev[it] } + listOf(1)
}

fun main() {
    pascal().take(10).forEach(::println)
}
```
![20200517-124137](http://cdn.51git.cn/2020-05-17-20200517-124137.png)

在这里有个小建议，可以关注一些你感兴趣的官方、大牛的 Twitter 账号，还有，他们不定时就会分享一些新的技术、新的文章等等。

### 官方、大牛的 Twitter 账号

* Jake Wharton @JakeWharton：Android 之神不需要过多介绍。
* Android @Android: 官网账号
* Marcin Moskala @marcinmoskala：是 「Effective Kotlin」一书的作者之一。「Effective Kotlin」总结了 Kotlin 社区的最佳实践和经验
* Arnaud Giuliani @arnogiu: Koin 软件工程师-演讲者-写技术博客-开源
* Kotlin @kotlin：官网账号
* Kt. Academy @ktdotacademy：Kt学院（原Kotlin学院）的任务是简化Kotlin学习
* MIT Tech Review @techreview：来自 MIT，质量不错的科技时评，关注最前沿的科技动态
* Andrew Ng @andrewYNg：Coursera 创始人，AI 大牛吴恩达
* JetBrains @jetbrains：IntelliJ IDEA、ReSharper、PyCharm、TeamCity、Kotlin等的创造者
* 还有很多很多 ......

以上大牛的都会在 medium 上学技术文章，有能力的朋友可以多从上面看最新的文章，国内也有很多资源，可以访问译者自己撸的导航网站，"[为互联网人而设计   国内国外名站导航](http://site.51git.cn/)" ，收集了国内外热门网址，涵括新闻、体育、生活、娱乐、设计、产品、运营、前端开发、Android 开发等等导航网站

![](http://cdn.51git.cn/2020-05-17-15893894983433.jpg)

## 参考文献

* [Item: Consider aggregating elements to a map](https://blog.kotlin-academy.com/item-consider-aggregating-elements-to-map-a5a35a7b6c61)
*  [Effective Java in Kotlin, item 7: Eliminate obsolete object references](https://blog.kotlin-academy.com/effective-java-in-kotlin-item-7-eliminate-obsolete-object-references-9a197b6fb728)
* [Improve Java to Kotlin code review](https://medium.com/@elye.project/improve-java-to-kotlin-code-review-61e7bbb2c663)

## 结语

致力于分享一系列 Android 系统源码、逆向分析、算法、翻译相关的文章，目前正在翻译一系列欧美精选文章，请持续关注，除了翻译还有对每篇欧美文章思考，如果对你有帮助，请帮我点个赞，感谢！！！期待与你一起成长。


