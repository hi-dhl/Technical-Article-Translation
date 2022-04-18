# Android 11 提高 App 冷启动速度 5% 以上


![](https://img.hi-dhl.com/16501815271369.jpg)

> Hi 大家好，我是 DHL。公众号：ByteCode ，专注分享最新技术原创文章，涉及 Kotlin、Jetpack、算法动画、数据结构、系统源码、 LeetCode / 剑指 Offer / 多线程 / 国内外大厂算法题等等。
> * 原文: [https://medium.com/improving-app...](https://medium.com/androiddevelopers/improving-app-startup-with-i-o-prefetching-62fbdb9c9020)
> * 译者：DHL
> * 本文已收录于仓库 [Technical-Article-Translation](https://github.com/hi-dhl/Technical-Article-Translation)

近一年多以来一直在做性能优化（ OOM、Native、ANR 等等），在后面我也会写一些性能相关的文章，将自己学习和实践所得分享出来。以今天这篇文章作为开端。

在 Android 11 上增加了一个新的功能 IORap，IORap 将会减少 App 冷启动耗时，经过在各种设备上测试，App 的启动速度（冷启动）平均提高了 5% 以上，部分设备提高了 20% 以上，开发者不需要做任何任何事情，即可享受带来的启动优化收益。

![](https://img.hi-dhl.com/15994022239932.jpg)

### IORap prefetching for Android apps

IORap 会提前预测需要那些 I/O 并将他们提前，通过这种方式减少 App 启动耗时。大量的 App 启动时间很长，是因为 blocking I/O 导致 IO 请求队列未到达饱和，在预取数据之后同时压缩 I/O ，App 可以很快的从 kernel pagecache 中访问预取数据，从而减少 App 启动耗时。

![](https://img.hi-dhl.com/16501036147840.jpg)

我们测试了在 Google Play Store 上一些热门的应用，80% 的 App 在启动期间，因为 blocking I/O 耗费了 10% 以上的时间，80% 的 App 耗费了 20% 以上的时间。我们在 Google Play Store 上测试了大部分应用都可以从 IORap 中获得收益。

![](https://img.hi-dhl.com/16501047850133.jpg)

![](https://img.hi-dhl.com/16501045308746.jpg)


IORap 作为一个独立的 service，它通过 IPC 与 package manager，activity manager， perfetto service 等等交互，以下是 IORap 的架构图。

![](https://img.hi-dhl.com/16501046084407.jpg)


* **Step 1: Collecting perfetto traces**

IORap 基于一定的策略分析预取  I/O ，通过 perfetto 进行跟踪记录，会在 kernel pagecache 中添加和删除的页面。经过测试，启动期间通过 perfetto 进行跟踪记录造成的开销可以忽略不计。


* **Step 2: Generating prefetch list**

基于上面的 perfetto trace，IORap 会在设备空闲时，生成预取列表，预取列表包含启动期间需要读取的文件信息（名称，偏移，长度）， IORap 会根据  perfetto trace 分析 mm_pagemap 事件，并将结果 (inode、偏移量、长度) 转换为 (名称、偏移量、长度)，然后将数据存储在预取列表中，预取列表是一个 protobuf 文件。

![](https://img.hi-dhl.com/16501062044171.jpg)

* **Step 3: I/O prefetching**

经过上一步，生成预取列表之后，后续运行 App 时 IORap 可以为 App 预取对应的数据，在上一步执行完之后，不在需要 perfetto trace, 开发者不需要做任何事情，系统会在用户点击图标时或者通过 Intent 请求它，执行预取操作，享受带来的启动优化。

* **Step 4: Obsoleting the prefetch list**

预取列表不会永久存在，会因为一些事件导致预取列表过时，而被删除，当 App 更新时，由于更新过程中可能会发生变化，和之前的预取数据会有一些差异，所以不建议在这个阶段预取数据，另外 `dexopt` 会在 App 安装后进行优化，优化后的 App，数据不会发生改变，这会使预取列表过时，过时的预取列表将被删除，这时会开始新一轮的 perfetto trace。

### Improvements & Observation


通过对比几个实验的结果，我们可以确定 IORap 对于低端机和高端机都会有收益，平均而言， IORAP 可以提高 26％ 的启动速度，对于启动期间有大量 I/O 的 App 会有很大的帮助，例如，Spotify 低端设备和高端设备有两位数字的优化效果。

![](https://img.hi-dhl.com/16501138399987.jpg)


在实验过程中，发现了一个现象 IORap 性能会受到预取数据的影响，跟踪持续时间对于 IORap 来说非常重要，跟踪持续时间越短，预取的数据就越少，获得的性能也越低。另一方面，长时间的预取会导致需要预取的数据过多，这可能会导致启动速度变慢，我们可以根据 `ReportFullyDrawn` 事件的时间戳来估计跟踪持续时间。在正确的调用 `reportFullyDrawn` 回调可以提高 IORap 的性能。

### Future Development

我们对 IORap 所表现出来的性能非常的兴奋，在未来将会朝着以下两方向进行优化。

* 保证性能的前提之下，更频繁地进行预取，如果预取可以在分析期间完成，那就更好了。通过提供一个预构建的预取列表，我们可以在生成预取列表之前消除一些性能差距
* IORap 可以预测应用启动，更早的开始预取，从而进一步加快 App 启动

### Conclusion

可以在 App 启动完成之后，调用 `reportFullyDrawn` 来帮助 IORap 进行更好的优化，IORap 主要有助于减少 I/O 阻塞时间，因此可以考虑对 App 启动进行分析，发现和解决其他可能存在的性能问题。


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

* [Stack Overflow 上最热门的 10 个 Kotlin 问题](https://mp.weixin.qq.com/s/rRa-EBAgFENurfVfYCrUbA)
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

