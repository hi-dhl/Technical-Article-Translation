# Android 13这些权限废弃，你的应用受影响了吗？

![](https://img.hi-dhl.com/permisstion_qr.png)


> hi 大家好，我是 DHL。公众号：ByteCode ，专注分享最新技术原创文章，涉及 Kotlin、Jetpack、算法动画、数据结构、系统源码、 LeetCode / 剑指 Offer / 多线程 / 国内外大厂算法题等等。
> 原文地址：[Android 13这些权限废弃，你的应用受影响了吗？](https://mp.weixin.qq.com/s/t_E6kOU2jTJ82pcJ7ZkdMA)

无论是更改个人头像、分享照片、还是在电子邮件中添加附件，选择和分享媒体文件是用户最常见的操作之一。在听取了 Android 用户反馈之后，我们对应用程序访问媒体文件的方式做了一些改变。

### Android 13 已被废弃的权限

许多用户告诉我们，文件和媒体权限让他们很困扰，因为他们不知道应用程序想要访问哪些文件。

在 Android 13 上废弃了 `READ_EXTERNAL_STORAGE` 和 `WRITE_EXTERNAL_STORAGE` 权限，用更好的文件访问方式代替这些废弃的 API。

从 Android 10 开始向共享存储中添加文件不需要任何权限。因此，如果你的 App 只在共享存储中添加文件，你可以停止在 Android 10+ 上申请任何权限。

在之前的系统版本中 App 需要申请  `READ_EXTERNAL_STORAGE`  权限访问设备的文件和媒体，然后选择自己的媒体选择器，这为开发者增加了开发和维护成本，另外 App 依赖于通过 `ACTION_GET_CONTENT` 或者 `ACTION_OPEN_CONTENT` 的系统文件选择器，但是我们从开发者那里了解到，它感觉没有很好地集成到他们的 App 中。


![System File Picker using ACTION_OPEN_CONTENT](https://img.hi-dhl.com/202210310029344.jpeg)


### 图片选择器

在 Android 13 中，我们引入了一个新的媒体工具 Android 照片选择器。该工具为用户提供了一种选择媒体文件的方法，而不需要授予对其整个媒体库的访问权限。

它提供了一个简洁界面，展示照片和视频，按照日期排序。另外在 "Albums" 页面，用户可以按照屏幕截图或下载等等分类浏览，通过指定一些用户是否仅看到照片或视频，也可以设置选择最大文件数量，也可以根据自己的需求定制照片选择器。简而言之，这个照片选择器是为私人设计的，具有干净和简洁的 UI 易于实现。

![](https://img.hi-dhl.com/202210310029345.gif)



我们还通过谷歌 Play 系统更新 (2022 年 5 月 1 日发布)，将照片选择器反向移植到 Android 11 和 12 上，以将其带给更多的 Android 用户。

开发一个照片选择器是一个复杂的项目，新的照片选择器不需要团队进行任何维护。我们已经在 `ActivityX 1.6.0` 版本中为它创建了一个 `ActivityResultContract`。如果照片选择器在你的系统上可用，将会优先使用照片选择器。

```kotlin
// Registering Photo Picker activity launcher with a max limit of 5 items
val pickMultipleVisualMedia = registerForActivityResult(PickMultipleVisualMedia(5)) { uris ->
    // TODO: process URIs
}
// Launching the photo picker (photos & video included)
pickMultipleVisualMedia.launch(PickVisualMediaRequest(PickVisualMedia.ImageAndVideo))
```

如果希望添加类型进行筛选，可以采用这种方式。

```kotlin
// Launching the photo picker (photos only)
pickMultipleVisualMedia.launch(PickVisualMediaRequest(PickVisualMedia.ImageOnly))
// Launching the photo picker (video only)
pickMultipleVisualMedia.launch(PickVisualMediaRequest(PickVisualMedia.VideoOnly))
// Launching the photo picker (GIF only)
pickMultipleVisualMedia.launch(PickVisualMediaRequest(PickVisualMedia.SingleMimeType("image/gif")))
```

可以调用 `isPhotoPickerAvailable` 方法来验证在当前设备上照片选择器是否可用。

### ACTION_GET_CONTENT 将会发生改变

正如你所见，使用新的照片选择器只需要几行代码。虽然我们希望所有的 Apps 都使用它，但在 App 中迁移可能需要一些时间。

这就是为什么我们使用 `ACTION_GET_CONTENT` 将系统文件选择器转换为照片选择器，而不需要进行任何代码更改，从而将新的照片选择器引入到现有的 App 中。

![](https://img.hi-dhl.com/202210310029346.gif)




### 针对特定场景的新权限

虽然我们强烈建议您使用新的照片选择器，而不是访问所有媒体文件，但是您的 App 可能有一个场景，需要访问所有媒体文件（例如图库照片备份）。对于这些特定的场景，我们将引入新的权限，以提供对特定类型的媒体文件的访问，包括图像、视频或音频。您可以在文档中阅读更多关于它们的内容。

如果用户之前授予你的应用程序 `READ_EXTERNAL_STORAGE` 权限，系统会自动授予你的 App 访问权限。否则，当你的 App 请求任何新的权限时，系统会显示一个面向用户的对话框。

所以您必须始终检查是否仍然授予了权限，而不是存储它们的授予状态。

![](https://img.hi-dhl.com/202210310029347.jpeg)


下面的决策树可以帮助您更好的浏览这些更改。

![](https://img.hi-dhl.com/202210310029348.jpeg)


我们承诺在保护用户隐私的同时，继续改进照片选择器和整体存储开发者体验，以创建一个安全透明的 Android 生态系统。

新的照片选择器被反向移植到所有 Android 11 和 12 设备，不包括 Android Go 和非 gms 设备。


<br/>

**全文到这里就结束了，感谢你的阅读，坚持原创不易，欢迎在看、点赞、分享给身边的小伙伴，我会持续分享原创干货！！！**


**真诚推荐你关注我，公众号：ByteCode ，持续分享硬核原创内容，Kotlin、Jetpack、性能优化、系统源码、算法及数据结构、动画、大厂面经。**

<br/>

___

我开了一个云同步编译工具（SyncKit），主要用于本地写代码，然后同步到远程设备，在远程设备上进行编译，最将编译的结果同步到本地，代码已经上传到 Github，欢迎前往仓库 [hi-dhl/SyncKit](https://github.com/hi-dhl/SyncKit)  查看。

* [仓库 SyncKit：https://github.com/hi-dhl/SyncKit](https://github.com/hi-dhl/SyncKit)
* [下载地址：https://github.com/hi-dhl/SyncKit/releases](https://github.com/hi-dhl/SyncKit/releases)


---
<div align="center">
    <table>
        <tr>
             <td><a href=" https://space.bilibili.com/498153238">哔哩哔哩</a></td>
             <td><a href=" https://juejin.im/user/2594503168898744">掘金</a></td>
             <td><a href=" https://hi-dhl.com">博客</td>
             <td><a href=" https://github.com/hi-dhl">Github</a></td>
         </tr>
     </table>
</div>

**近期必读热门文章**

* [Android 利器，我开发了云同步编译工具](https://mp.weixin.qq.com/s/jRlrtnOg6Ww-C_Xb6719EA)
* [Twitter 上有趣的代码](https://mp.weixin.qq.com/s/ExxJMyYZP3sd9pnvdiEFAg)
* [谁动了我的内存，揭秘 OOM 崩溃下降 90% 的秘密](https://mp.weixin.qq.com/s/QZ6zZ6GoNpB5MWGi31k-UA)
* [反射技巧让你的性能提升 N 倍](https://mp.weixin.qq.com/s/AgEr6GhylkUfG_zWE5LTOw)
* [90%人不懂的泛型局限性，泛型擦除，星投影](https://mp.weixin.qq.com/s/brNq5OxitQ7WhPbA9y5lrA)
* [90%的人都不知道的知识点，Kotlin 和 Java 的协变和逆变](https://mp.weixin.qq.com/s/TqX2cLJFBe-NtxyGdGeYow)
* [揭秘反射真的很耗时吗，射 10 万次耗时多久](https://mp.weixin.qq.com/s/Ah8Yau_UW07s6LnGjrG4hA)
* [Android 12 已来，你的 App 崩溃了吗？](https://mp.weixin.qq.com/s/NuqAYoUq_0OorM1rVHUEHA)
* [Google 宣布废弃 LiveData.observe 方法](https://mp.weixin.qq.com/s/fp1ZOmqAcEBv2f7ec1r-zw)
* [影响性能的 Kotlin 代码（一）](https://mp.weixin.qq.com/s/8dAbt1-mcCVLWLXKC-1_xw)
* [揭秘 Kotlin 中的 == 和 ===](https://mp.weixin.qq.com/s/sYj_-wqENr9Jaw1p8iP4Jg)


**开源新项目**


* 云同步编译工具（SyncKit），本地写代码，远程编译，欢迎前去查看 [SyncKit](https://github.com/hi-dhl/SyncKit)

* KtKit 小巧而实用，用 Kotlin 语言编写的工具库，欢迎前去查看 [KtKit](https://github.com/hi-dhl/KtKit)

* 最全、最新的 AndroidX Jetpack 相关组件的实战项目以及相关组件原理分析文章，正在逐渐增加 Jetpack 新成员，仓库持续更新，欢迎前去查看 [AndroidX-Jetpack-Practice](https://github.com/hi-dhl/AndroidX-Jetpack-Practice)

* LeetCode / 剑指 offer，包含多种解题思路、时间复杂度、空间复杂度分析，[在线阅读](https://leetcode.hi-dhl.com)



