# 0xT08 [译]官宣 有趣的 Android Studio 版本新方案


![AndroidStudio3.001](http://img.hi-dhl.com/AndroidStudio3.001.png)


> * 原标题：Announcing Android Studio Arctic Fox (2020.3.1) & Android Gradle plugin 7.0
> * 原地址：[https://android-developers......](https://android-developers.googleblog.com/2020/12/announcing-android-studio-arctic-fox.html?m=1)
> * 原作者：Google
> * 译者：hi-dhl
> * 公众号：ByteCode，致力于分享最新技术原创文章，涉及 Kotlin、Jetpack、算法、译文、系统源码相关的文章

Android Studio 版本命名新方案，带来的好处是升级 Android Studio 不需要同时升级 Gradle 插件，有趣的是以动物的名字来命名，全文分为**译文**和**译者思考**两部分。

## 译文

于 2020.12.1 号 发布了  Android Studio Arctic Fox(2020.3.1) canary 版本，以及 Android Gradle 插件(AGP) 7.0.0-alpha01 版本，在新版本中，我们调整了 Android Studio 和 Gradle 插件的版本方案，这一变化将 Gradle 插件从 Android Studio 版本中分离出来，这样更加清楚的知道 Android Studio 在每个版本中使用的 IntelliJ 版本。

**Android Studio 新的版本方案**

随着 Android Studio Arctic Fox(2020.3.1) 的推出，我们将使用更接近 IntelliJ IDEA (Android Studio 所基于的IDE) 的基于年的版本方案，我们正在改变版本命名方案，同时也添加了一些重要的属性：年份， IntelliJ 的版本，加入了 feature 和 patch 级别。通过名字的改变，你可以很快地知道 Android Studio 使用的 IntelliJ 的版本号。此外，每个主要版本都有一个规范的代号，从 Arctic Fox 开始，然后按字母顺序进行，以方便查看哪个版本是最新的。


我们建议您使用最新的 Android Studio，以便可以使用最新功能和质量改进。 为了使更新更容易，我们对版本进行了更改，将 Android Studio 与 Gradle 插件版本分离。 要记住的一个重要细节是，更新 IDE 的时候，对构建系统编译和打包应用程序的方式没有影响。 相反，应用程序构建过程的更改和 APK/Bundles 由项目 AGP 版本决定。 

因此，即使在开发周期的后期，也可以安全地更新 Android Studio 版本，因为项目中使用的 AGP 版本可以和 Android Studio 版本不同。 

最后，在新的版本系统中，只要您将 AGP 版本保持在稳定版本，可以很方便的在项目中同时运行 Android Studio 的稳定版本和预览版本。

按照以前的版本命名方案，这个版本是 Android Studio 4.3，但是在新的版本方案中，这个版本是 Android Studio Arctic Fox (2020.3.1) Canary 1 或者 Arctic Fox。

![](http://img.hi-dhl.com/16070401948489.jpg)

接下来，我们将介绍 Android Studio 新的版本命名方案。

`<Year of IntelliJ Version>.<IntelliJ major version>.<Studio major version>`

* 前两个数字代表 Android Studio 使用的 IntellIj 的版本号，对于这个版本，是 2020.3
* 第三个数字表示 Android Studio 主要的版本，从 1 开始递增
* 为了更容易引用每个版本，我们给主要版本起了一个名称，根据动物名称从 a 开始递增到 Z。最初发行的名字是 Arctic Fox

**Android Gradle 插件版本新方案**

在 AGP 7.0.0 中，我们采用了 [semantic versioning](https://semver.org/) 的原则，并与 AGP 要求的 Gradle 版本保持一致，Android Studio 和  Gradle 插件之间的兼容性保持不变。使用 AGP 稳定版本的项目可以使用最新的 Android Studio 版本打开。

我们将会发布另外一篇文章，详细介绍 AGP 版本控制方案和 AGP 7.0 的新特性。

## 译者思考

对于新版本的命名方案最大的好处就是，升级 Android Studio 不需要同时升级 Gradle 插件，也就意味着，只要将 Gradle 插件的版本保持在稳定版本，可以很方便的在项目中同时运行 Android Studio 的稳定版本和预览版本。

但是新的 Gradle 插件会带来一些新特性，所以升级 Android Studio 版本的同时，可以同时升级 Gradle 插件的版本到稳定版本，我们来汇总一下 Android Studio 之前的命名方案 和  Gradle 插件的对应关系。


**Android studio 插件版本与 gradle 版本对应关系如下所示：**

| AS 插件版本 |  Gradle 版本 |
| --- | --- |
| 1.0.0 - 1.1.3 | 2.2.1 - 2.3 |
| 1.2.0 - 1.3.1 | 2.2.1 - 2.9 |
| 1.5.0 | 2.2.1 - 2.13 |
| 2.0.0 - 2.1.2 | 2.10 - 2.13 |
| 2.1.3 - 2.2.3 | 2.14.1+ |
| 2.3.0+ | 3.3+ |
| 3.0.0+ | 4.1+ |
| 3.1.0+ | 4.4+ |
| 3.2.0 - 3.2.1 | 	4.6+ |
| 3.3.0 - 3.3.3 | 	4.10.1+ |
| 3.4.0 - 3.4.3 | 	5.1.1+ |
| 3.5.0 - 3.5.4 | 	5.4.1+ |
| 3.6.0 - 3.6.4 | 	5.6.4+ |
| 4.0.0+ | 	6.1.1+ |
| 4.1.0+ | 	6.5+ |

以上信息参考 [Android Gradle 插件版本说明](https://developer.android.google.cn/studio/releases/gradle-plugin#updating-plugin)

Android Studio 根据动物名称来命名，而 Android 系统 10.0 之前都是以甜点的方式来命名，我们在来会汇总一下 Android 系统的命名方案。

2007 年 11 月 5 日发布最初的版本（Android 0.5），至今 Android 发行了多个版本，Android 操作系统有预发行的内部版本，分别为铁臂阿童木（Astro）与机器人班亭（Bender），从 2009 年 5 月开始， Android 的版本代号以甜点来命名，且每个代号间的前缀以英文本母序接续排列。

**Android 系统名字、版本、API level 的对应关系如下所示：**

| 名称 | 版本号 | 发版日期 | API |  API | 
| --- | --- | --- | --- | --- |
| Android 1.0 | 1.0 | 2008年9月23日 | 1 | BASE |
| Android 1.1 | 1.1 | 2009年2月9日  | 2 | 	BASE_1_1 |
| Android Cupcake（纸杯蛋糕） | 1.5 | 2009年4月27日 | 3 | CUPCAKE |
| Android Donut（甜甜圈） | 1.6 | 2009年9月15日 | 4 | DONUT |
| Android Eclair（闪电泡芙） | 2.0 – 2.1 | 2009年10月26日 | 5 – 7 | ECLAIR_MR1（2.1.x）<br/> ECLAIR_0_1（2.0.1）<br/> ECLAIR（2.0） |
| Android Froyo（优格冰淇淋） | 2.2 – 2.2.3 | 2010年5月20日 | 8 | FROYO |
| Android Gingerbread（姜饼） | 2.3 – 2.3.7 | 2010年12月6日 | 9 - 10 | GINGERBREAD_MR1（ 2.3.3 -  2.3.4）<br/> GINGERBREAD（2.3、2.3.1、2.3.2）|
| Android Honeycomb（蜂巢） | 3.0 – 3.2.6 | 2011年2月22日 | 11 - 13 | HONEYCOMB_MR2（3.2）<br/> HONEYCOMB_MR1（3.1x） <br/> HONEYCOMB（3.0.x）|
| Android Ice Cream Sandwich（冰淇淋三明治） | 4.0 – 4.0.4 | 2011年10月18日 | 14 - 15 | ICE_CREAM_SANDWICH_MR1（4.0.3、4.0.4）<br/>  ICE_CREAM_SANDWICH （4.0、4.0.1、4.0.2）|
| Android Jelly Bean（果冻豆） | 4.1 – 4.3.1 | 2012年7月9日 | 16 – 18 | JELLY_BEAN_MR2（4.3） <br/> JELLY_BEAN_MR1（4.2 - 4.2.2）<br/> JELLY_BEAN（4.1 - 4.1.1）|
| Android KitKat（奇巧巧克力） | 4.4 – 4.4.4 | 2013年10月31日 | 19 - 20 | KITKAT |
| Android Lollipop（棒棒糖） | 5.0 – 5.1.1 | 2014年11月12日 | 21 - 22 | LOLLIPOP_MR1（5.1） <br/> LOLLIPOP（5.0）|
| Android Marshmallow（棉花糖） | 6.0 – 6.0.1 | 2015年10月5日 | 23 | M |
| Android Nougat（牛轧糖） | 7.0 – 7.1.2 | 2016年8月22日 | 24 - 25 | N_MR1（7.1 - 7.11） N（7.0）|
| Android Oreo（奥利奥） | 8.0 – 8.1 | 2017年8月21日 | 26 – 27 | O_MR1（8.1）  O （8.0）|
| Android Pie（派） | 9 | 2018年8月6日 | 28 | P |
| Android 10 | 10 | 2019年9月3日 | 29 | Q |
| Android 11 | 11 | 2020年2月19日 | 30 | R |

从 Android Q 开始不再以甜品命名，且直接称 Android Q 为 Android 10。以上信息参考 [uses-sdk](https://developer.android.com/guide/topics/manifest/uses-sdk-element.html)

## 结语

全文到这里就结束了，如果有帮助 **点个赞** 就是对我最大的鼓励！


![](http://img.hi-dhl.com/16070540786006.jpg)

---

最后推荐我一直在更新维护的项目和网站：

* 全新系列视频：现代 Android 开发 (MAD) 技巧系列教程：[在线查看](https://madskills.hi-dhl.com)

* 计划建立一个最全、最新的 AndroidX Jetpack 相关组件的实战项目 以及 相关组件原理分析文章，正在逐渐增加 Jetpack 新成员，仓库持续更新，欢迎前去查看：[AndroidX-Jetpack-Practice](https://github.com/hi-dhl/AndroidX-Jetpack-Practice)

* LeetCode / 剑指 offer / 国内外大厂面试题 / 多线程 题解，语言 Java 和 kotlin，包含多种解法、解题思路、时间复杂度、空间复杂度分析<br/>

    <image src="http://cdn.51git.cn/2020-10-04-16017884626310.jpg" width = "500px"/>
  
    * 剑指 offer 及国内外大厂面试题解：[在线阅读](https://offer.hi-dhl.com)
    * LeetCode 系列题解：[在线阅读](https://leetcode.hi-dhl.com)

* 最新 Android 10 源码分析系列文章，了解系统源码，不仅有助于分析问题，在面试过程中，对我们也是非常有帮助的，仓库持续更新，欢迎前去查看 [Android10-Source-Analysis](https://github.com/hi-dhl/Android10-Source-Analysis)

* 整理和翻译一系列精选国外的技术文章，每篇文章都会有**译者思考**部分，对原文的更加深入的解读，仓库持续更新，欢迎前去查看 [Technical-Article-Translation](https://github.com/hi-dhl/Technical-Article-Translation)

* 「为互联网人而设计，国内国外名站导航」涵括新闻、体育、生活、娱乐、设计、产品、运营、前端开发、Android 开发等等网址，欢迎前去查看 [为互联网人而设计导航网站](https://site.51git.cn)




