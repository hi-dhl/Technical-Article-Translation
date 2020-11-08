# 0xT7 [译] Google 官方正解是否应该学习 Kotlin

![](http://img.hi-dhl.com/kotlin-vs-java.030.png)


> * 原标题：Should I learn Kotlin for Android and other FAQs
> * 原地址：[https://medium.com/androiddevelopers...](https://medium.com/androiddevelopers/should-i-learn-kotlin-for-android-and-other-faqs-88a2bb281a60)
> * 原作者：Florina Muntenescu
> * 译者：hi-dhl

这篇文章来自 Google 开发技术推广工程师 `Florina Muntenescu` 分享的一篇文章，在这篇文章里精选了 Google 宣布支持 Kotlin 以来最热门的几个问题，回答了是否应该学习 Kotlin、以及如何学习 Kotlin。全文分为 **译文** 和 **译者思考** 两个部分。

## 译文

自从我们在 2017 年宣布支持 Kotlin 以来，收到了很多关于 Kotlin 在 Android 上的问题：如何学习它 Kotlin，如何在 App 上使用 Kotlin，有什么好的 Kotlin 学习教程，以及 Google 内部是否在使用 Kotlin，和未来对 Java 语言的规划是什么，将通过这篇文章来回答这些问题。

**Q: 是否应该学习 Kotlin ?**

我们收到了很多类似的问题，总结如下：

* 对于 Android 初学者，应该先学习 Java 还是 Kotlin？
* 如果已经掌握了 Java 基础知识，现在应该切换到 Kotlin 开发 Android 吗？
* 对于 Java 开发人员，如果想要学习 Android，建议先学习 Kotlin 还是 Java?

用一句简短的话，回答上面的问题：**是的，开始学习和使用 Kotlin** 。

下面会用很长的文字来分析，为什么应该学习和使用 Kotlin。

### Kotlin & Android

2017年，我们在 Google I / O 上宣布支持 Kotlin，从那个时候我们已经开始准备关于 Kotlin 的 API、文档、案例，在 2019 年 Kotlin 成为 Android 的首选语言之后，我们开始更加依赖 Kotlin 的特性，例如，我们推荐使用协程执行异步任务。

**Kotlin-first libraries**

首先我们在 Jetpack(Room、LiveData、ViewModel 和 WorkManager) 中添加了协程的支持，从而改变了在 Android 上执行异步操作的方式，Firebase Android SDK 和大量的 Jetpack 库都用到了 [Kotlin extension libraries](https://developer.android.com/kotlin/ktx) (KTX)。

现在很多库例如 Paging 3.0 和 DataStore 首先使用 Kotlin 开发的。[Jetpack Compose](https://developer.android.com/jetpack/compose) 是我们新的、未绑定的声明式 UI 工具包，它也是使用 Kotlin 开发的。

**Tooling**

开发的效率来源于强大的工具。因此，我们对 Kotlin 编译工具做了很多改进，包括对 Kotlin JVM 编译器、Kotlin 的 R8 优化，甚至开发了 [Kotlin Symbol Processing](https://github.com/google/ksp)。我们添加了 Android Kotlin Live 模板，会自动在 App 中添加一些通用模块，而且新的 Kotlin 的 Lint 检查工具可以帮助你检查 Kotlin 语言规范，当您将代码从 Java 转到 Kotlin 的时候，这个工具特别有用。


### Q: Google 内部是否在使用 Kotlin

在 Google 内部我们也在推荐使用 Kotlin，我们有超过 60 个应用（例如：Home、Drive、Maps 等等） 已经开始使用 Kotlin 开发了，到目前为止，在我们的代码库中有超过 **200 万行 Kotlin 代码**。


### Q: 是否应该将 App 迁移到 Kotlin？

我们收到了很多类似的问题，但是是否迁移到 Kotlin 取决于你，如果您对当前的代码库和技术栈感到满意，并且熟练地使用您的解决方案来管理异步任务，并且有一个有效的方法来捕获错误，迁移到 Kotlin 可能不是一个很好的方案。

如果你喜欢 Kotlin，而且想使用最新的 Jetpack API，那么你应该考虑将 Kotlin 加入到你的 App 中，Kotlin 优点之一是它与 Java 有很好的互操作性，您可以在项目中逐步使用它，可能首先在测试用例中使用 Kotlin，然后在新功能中使用 Kotlin，当熟悉之后，可以尝试将 Java 代码转为 Kotlin 代码。

如果想要迁移到 Kotlin , 可以查看我们的教程 [Converting to Kotlin codelab](https://codelabs.developers.google.com/codelabs/java-to-kotlin#0)。

### Q: 在 Android 中使用 Java 怎么样？

Kotlin 会编译成 Java 代码，它们是可以共存的，我们喜欢 Kotlin 因为使用它编写代码的更简洁、也更加安全，同时我们也会继续支持 Java 语言。例如，在 Android 11 中，我们增加了 OpenJDK 13 一系列 API 的支持，而 Android Studio 也允许你在所有 Android 设备上使用其中一些 API，无论操作系统版本是什么, 点击 [这里](https://medium.com/androiddevelopers/support-for-newer-java-language-apis-bca79fc8ef65) 阅读更多新的 API。

### Q: 学习 Kotlin 的最好方法是什么?

切换到一种新的语言不是一件容易的事，但是我们也正努力使它变得更容易。

* 点击 [courses](https://developer.android.com/kotlin/campaign/learn) 开始学习 Kotlin，针对所有级别的开发者，从初级到高级所有课程，这些课程将帮助你提高在 Android 中使用 Kotlin，[Android Basics in Kotlin](https://developer.android.com/courses/android-basics-kotlin/course) 这是给没有经验的人提供的一个新的在线课程，还有一些高级课程教你如何使用协程。
* 所有的文档都包含了 Kotlin 代码片段，可以非常方便的比较两种语言，而且所有示例都有 Kotlin 版本。
* 你可以查看 [文章](http://goo.gle/kotlin-posts) 和 [视频](http://goo.gle/kotlin-videos) 来学习如何使用 Kotlin。
* 对于开发者或者团队想要切换到 Kotlin 我们提供一些指导，可以查看网页 [developers.android.com/kotlin](https://medium.com/androiddevelopers/should-i-learn-kotlin-for-android-and-other-faqs-88a2bb281a60)


宣布支持 Kotlin 到现在已经三年了，我们一直在努力支持 Kotlin 和这个生态，与 JetBrains 一起为 Kotlin 建立了一个基础，以确保该语言能够很好的使用。不仅仅如此而已，在 Google 内部有一个团队专门研究 Kotlin 编译器，我们正在构建的 Jetpack API 不仅仅支持 Kotlin，但是会优先支持 Kotlin，我们也在努力让 Kotlin 在 Android 上的体验更好。

## 译者思考

自从 Google 宣布 Kotlin 成为 Android 开发的首选语言开始，Google 一直致力于让 Kotlin 变得更加的简单。

在 Kotlin 之初有个非常著名的库 Anko，Anko 是 JetBrains 开发的一个非常强大的库，它主要的目的是替代以前 XML 的方式，使用代码生成 UI 布局，并且封装了一系列工具，帮助开发者快速的使用 Kotlin，简化了 Kotlin 在 Android 上的使用，它有好几个扩展库：

* Anko Commons：一个轻量级的库，包含了一些通用功能  intents、 dialogs、 logging 等等
* Anko Layouts： 替代以前 XML 的方式，使用代码生成 UI 布局
* Anko SQLite：简化了 SQLite 的使用
* Anko Coroutines：基于 [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines) 开发，简化了协程的使用

Anko 是非常成功的项目，它的出现让 Kotlin 在 Android 上的体验更好，但是遗憾的是 **在 2019 年的时候这个库已经不在维护，不在维护，不在维护** ，因为自从 Google 宣布支持 Kotlin，让 Kotlin 成为 Android 开发的首选语言开始，Google 开发了很多库使得 Kotlin 在 Android 上体验更好，完全可以替代 Anko 的各个部分，所以 Anko 团队宣布不在维护了。

* Android KTX： 是 Kotlin 扩展库，封装了一系列工具，简化了 Kotlin 的使用
* Jetpack Compose ： 替代以前 XML 的方式，用于构建原生 UI 的工具
* Room：Google 提供的 ORM 框架，简化了 SQLite 的使用
* Flow：flow 是对 Kotlin 协程的扩展，让我们可以像运行同步代码一样运行异步代码，简化了对协程的使用，而且功能非常强大

### MAD Skills

不仅仅如此 Google 近期发布了 MAD Skills（Modern Android Development）新系列教程，旨在帮助开发者使用最新的技术，开发更好的应用程序，以视频和文章形式介绍 MAD 各个部分，包括 Kotlin、Android Studio、Jetpack、App Bundles 等等, Google 仅仅提供了视频和文章，我在这基础上，我做了一些扩展：

* 视频上添加上了中英文字幕，帮助更好的学习新技术
* 将会提供对应的实战案例，与视频一一对应
* 除了实战案例，还会提供对应的源码分析

每隔几个星期 Google 会发布一系列教程，目前已经开始了一系列关于导航组件 (Navigation component) 的视频教程。

双语视频已经同步到 GitHub 仓库 [MAD-Skills](https://github.com/hi-dhl/MAD-Skills) 可以先看视频部分，文章以及案例正在火速赶来，点击[在线查看](https://madskills.hi-dhl.com)。

除此之外，我还写了 Kotlin 和 Jetpack 系列文章，并且提供了对应的实战案例。

### Kotlin 系列

* [为数不多的人知道的 Kotlin 技巧以及 原理解析（一）](https://juejin.im/post/5edfd7c9e51d45789a7f206d)
* [为数不多的人知道的 Kotlin 技巧以及 原理解析（二）](https://juejin.im/post/6847902224467623950)
* [Kotlin StateFlow 搜索功能的实践 DB + NetWork](https://juejin.im/post/6876990111113248775)
* [Kotlin Sealed 是什么？为什么 Google 都用](https://juejin.im/post/6859980718588575757)
* [如何在项目中封装 Kotlin + Android Databinding](https://juejin.im/post/5e9c434a51882573663f6cc6)
* [放弃 Dagger 拥抱 Koin](https://juejin.im/post/6844904158324064269)
* [Kotlin 的性能优化那些事](https://juejin.im/post/5ec0f3afe51d454db11f8a94)
* [Kotlin 新秀 Coil VS Glide and Picasso](https://juejin.im/post/5edd1f5ae51d45789e0d9a22)

### Jetpack 系列

* [Jetpack 成员 App Startup 实践及原理分析](https://mp.weixin.qq.com/s/QRm5ByF6PqedWykFmvfb6A)
* [Jetpack 成员 Paging3 实践以及源码分析（一）](https://juejin.im/post/5ee998e8e51d4573d65df02b)
* [Jetpack 成员 Paging3 网络实践及原理分析（二）](https://juejin.im/post/5eeefbf4e51d45742c53ddce)
* [Jetpack 成员 Paging3 获取网络分页数据并更新到数据库中（三）](https://mp.weixin.qq.com/s/ABtWRiiIKlQe32Z6sFPbng)
* [Jetpack 成员 Hilt 实践（一）启程过坑记](https://mp.weixin.qq.com/s/6yqsc6P4aUn5alvzf9PPVg) 
* [Jetpack 成员 Hilt 结合 App Startup（二）进阶篇）进阶篇](https://mp.weixin.qq.com/s/-B7pCNFncGaBW3AJEy8zWA)
* [Jetpack 成员 Hilt 与 Dagger 区别 (三) 落地篇](https://mp.weixin.qq.com/s/VKyyNqAPFnlclGKnIbisAw) 
* [全方面分析 Hilt 和 Koin 性能](https://mp.weixin.qq.com/s/PsiCIOiV8FWVQ9HF8EvJ8w) 
* [神奇宝贝(PokemonGo)  眼前一亮的 Jetpack + MVVM 极简实战](https://juejin.im/post/6850037271253483534) 
* [Google 推荐在 MVVM 架构中使用 Kotlin Flow](https://juejin.im/post/6854573211930066951)
* [再见吧 buildSrc, 拥抱 Composing builds 提升 Android 编译速度](https://mp.weixin.qq.com/s/3bPwYt01BG2bGD3e84K-Gw)
* [Fragment 新特性 : Fragment Result API 使用以及源码分析](https://mp.weixin.qq.com/s/qL8UR2Akz_AIfV1khLIfjg) 
* [Google 建议使用这些 Fragment 的新特性](https://mp.weixin.qq.com/s/AVHkgYX8OVzbF2nxwQJ3qA)
* [再见 SharedPreferences 拥抱 Jetpack DataStore (一)](https://juejin.im/post/6881442312560803853)
* [再见 SharedPreferences 拥抱 Jetpack DataStore (二)](https://mp.weixin.qq.com/s/tT7TxUt2IMC4UVwuwMCBrA)

### Android 10 系列

* [0xA01 Android 10 源码分析：APK 是如何生成的](https://juejin.im/post/5e4366c3f265da57397e1189)
* [0xA02 Android 10 源码分析：APK 的安装流程](https://juejin.im/post/5e5a1e6a6fb9a07cb427d8cd)
* [0xA03 Android 10 源码分析：APK 加载流程之资源加载](https://juejin.im/post/5e6c8c14f265da574b792a1a)
* [0xA04 Android 10 源码分析：APK 加载流程之资源加载（二）](https://juejin.im/post/5e7f0f2c51882573c4676bc7)
* [0xA05 Android 10 源码分析：Dialog 加载绘制流程以及在 Kotlin、DataBinding 中的使用](https://juejin.im/post/5e9199db6fb9a03c7916f635)
* [0xA06 Android 10 源码分析：WindowManager 视图绑定以及体系结构](https://juejin.im/post/5ead0b865188256d545fd2f8)
* [0xA07 Android 10 源码分析：Window 的类型 以及 三维视图层级分析](https://juejin.im/post/5ed98f6df265da770f52035c)
* [更多](https://github.com/hi-dhl/Android10-Source-Analysis)

### GitHub 仓库以及 网站

* 计划建立一个最全、最新的 AndroidX Jetpack 相关组件的实战项目 以及 相关组件原理分析文章，正在逐渐增加 Jetpack 新成员，仓库持续更新，欢迎前去查看：[AndroidX-Jetpack-Practice](https://github.com/hi-dhl/AndroidX-Jetpack-Practice)

* LeetCode / 剑指 offer / 国内外大厂面试题 / 多线程 题解，语言 Java 和 kotlin，包含多种解法、解题思路、时间复杂度、空间复杂度分析<br/>

    <image src="http://cdn.51git.cn/2020-10-04-16017884626310.jpg" width = "500px"/>
  
    * 剑指 offer 、多线程、国内外大厂面试题：[在线阅读](https://offer.hi-dhl.com)
    * LeetCode 系列题解：[在线阅读](https://leetcode.hi-dhl.com)

* 最新 Android 10 源码分析系列文章，了解系统源码，不仅有助于分析问题，在面试过程中，对我们也是非常有帮助的，仓库持续更新，欢迎前去查看 [Android10-Source-Analysis](https://github.com/hi-dhl/Android10-Source-Analysis)

* 整理和翻译一系列精选国外的技术文章，每篇文章都会有**译者思考**部分，对原文的更加深入的解读，仓库持续更新，欢迎前去查看 [Technical-Article-Translation](https://github.com/hi-dhl/Technical-Article-Translation)

* 「为互联网人而设计，国内国外名站导航」涵括新闻、体育、生活、娱乐、设计、产品、运营、前端开发、Android 开发等等网址，欢迎前去查看 [为互联网人而设计导航网站](https://site.51git.cn)






