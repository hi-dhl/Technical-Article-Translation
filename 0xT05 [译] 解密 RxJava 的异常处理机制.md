# 0xT05 [译] 解密 RxJava 的异常处理机制

## 前言

> * 原标题:  Beyond Basic RxJava Error Handling
> * 原文地址: [https://proandroiddev.com/beyond-basic......](https://proandroiddev.com/beyond-basic-rxjava-error-handling-b0875d3877e0)
> * 原文作者：Elye，仅在 medium 上就有 5.2k+ 粉丝

今天看到一篇大神 Elye 关于 RxJava 异常的处理的文章，让我对 RxJava 异常的处理有了一个清晰的了解，用了 RxJava 很久了对里面的异常处理机制一直很懵懂。

**通过这篇文章你将学习到以下内容，将在译者思考部分会给出相应的答案**

* just 和 fromCallable 区别？
* 什么是 RxJavaPlugins.setErrorHandler？
* Crashes 发生在 just() 中的处理方案？
* Crashes 发生在 subscribe success 中的处理方案？
* Crashes 发生在 subscribe error 中的处理方案？
* Crashes 发生在 complete 中的处理方案？
* Crashes 发生在 complete 之前的处理方案？

这篇文章涉及很多重要的知识点，请耐心读下去，应该可以从中学到很多技巧。


## 译文

大部分了解 RxJava 的人都会喜欢它，因为它能够封装 onError 回调上的错误处理，如下所示：

```
Single.just(getSomeData())
 .map { item -> handleMap(item) } 
 .subscribe(
        { result -> handleResult(result) }, 
        { error -> handleError(error) } // Expect all error capture 
    )
```

你可能会以为所有的 Crashes 都将调用 handleError 来处理，但其实并全都是这样的

### Crashes 发生在 just() 中

我先来看一个简单的例子，假设 crashes 发生在 getSomeData() 方法内部

```
Single.just(getSomeData() /**🔥Crash🔥**/ )
   .map { item -> handleMap(item) } 
   .subscribe(
        { result -> handleResult(result) }, 
        { error -> handleError(error) } // Crash NOT caught ⛔️ 
    
```

这个错误将不会在 handleError 中捕获，因为 just() 不是 RxJava 调用链的一部分，如果你想捕获它，你可能需要在最外层添加 try-catch 来处理，如下所示：

```
try { 
   Single.just(getSomeData() /**🔥Crash🔥**/ )
      .map { item -> handleMap(item) } 
      .subscribe(
          { result -> handleResult(result) }, 
          { error -> handleError(error) } // Crash NOT caught ⛔️ 
      )
} catch (exception: Exception) {
   handleError(exception)  // Crash caught ✅
}
```

如果你不使用 just ，而是使用 RxJava 内部的一些东西，例如 fromCallable，错误将会被捕获

```
Single.fromCallable{ getSomeData() /**🔥Crash🔥**/ }
   .map { item -> handleMap(item) } 
   .subscribe(
        { result -> handleResult(result) }, 
        { error -> handleError(error) } // Crash caught ✅
    )
```

### Crashes 发生在 subscribe success 中

让我们来假设一下 Crashes 出现在 subscribe success 中，如下所示

```
Single.just(getSomeData() )
   .map { item -> handleMap(item) } 
   .subscribe(
        { result -> handleResult(result) /**🔥Crash🔥**/ }, 
        { error -> handleError(error) } // Crash NOT caught ⛔️ 
    )
```

这个错误将不会被 handleError 捕获，奇怪的是，如果我们将 Single 换成 Observable，异常就会被捕获，如下所示：

```
Observable.just(getSomeData() )
   .map { item -> handleMap(item) } 
   .subscribe(
        { result -> handleResult(result) /**🔥Crash🔥**/ }, 
        { error -> handleError(error) }, // Crash caught ✅
        { handleCompletion() }
    )
```

原因是在 Single 中成功的订阅被认为是一个完整的流。因此，错误不再能被捕获。而在 Observable 中，它认为 onNext 需要处理，因此 crash 仍然可以被捕获，那么我们应该如何解决这个问题呢

**错误的处理方式，像之前一样，在最外层使用 try-catch 进行异常捕获**

```
try { 
   Single.just(getSomeData())
      .map { item -> handleMap(item) } 
      .subscribe(
          { result -> handleResult(result)/**🔥Crash🔥**/ }, 
          { error -> handleError(error) } // Crash NOT caught ⛔️ 
      )
} catch (exception: Exception) {
   handleError(exception)  // Crash NOT caught ⛔️
}
```

但是这样做其实异常并没有被捕获，crash 依然在传递，因为 RxJava 在内部处理了 crash，并没有传递到外部

**一种很奇怪的方式，在 subscribe successful 中，执行 try-catch**

```
Single.just(getSomeData() )
   .map { item -> handleMap(item) } 
   .subscribe(
        { result -> try {
                handleResult(result) /**🔥Crash🔥**/
             } catch (exception: Exception) {
                handleError(exception) // Crash caught ✅            
             }
        }, 
        { error -> handleError(error) }, // Crash NOT caught ⛔️
    )
```

这种方式虽然捕获住了这个异常，但是 RxJava 并不知道如何处理

**一种比较好的方式**

上文提到了使用 Single 在 subscribe successful 中不能捕获异常，因为被认为是一个完整的流，处理这个情况比较好的方式，可以使用 doOnSuccess 方法

```
Single.just(getSomeData() )
   .map { item -> handleMap(item) }
   .doOnSuccess { result -> handleResult(result) /*🔥Crash🔥*/ }, }
   .subscribe(
        { /** REMOVE CODE **/ },
        { error -> handleError(error) } // Crash caught ✅
   )
```

当我们按照上面方式处理的时候，错误将会被 onError 捕获，如果想让代码更好看，可以使用 doOnError 方法，如下所示:

```
Single.just(getSomeData() )
   .map { item -> handleMap(item) }
   .doOnSuccess { result -> handleResult(result) /*🔥Crash🔥*/ }, }
   .doOnError { error -> handleError(error) } // Crash NOT stop ⛔️   
   .subscribe()
```

但是这并没有完全解决 crash 问题，虽然已经捕获了但并没有停止，因此 crash 仍然发生。

更准确的解释，它实际上确实捕获了 crash，但是 doOnError 不是完整状态，因此错误仍应该在 onError 中处理，否则它会在里面 crash，所以我们至少应该提供一个空的 onError

```
Single.just(getSomeData() )
   .map { item -> handleMap(item) }
   .doOnSuccess { result -> handleResult(result) /*🔥Crash🔥*/ }, }
   .doOnError { error -> handleError(error) } // Crash NOT stop ⛔️   
   .subscribe({} {}) // But crash stop here ✅
```

### Crashes 发生在 subscribe error 中

我们来思考一下如果 Crashes 发生在 subscribe error 中怎么处理，如下所示：

```
Single.just(getSomeData() )
   .map { item -> handleMap(item) } 
   .subscribe(
        { result -> handleResult(result) }, 
        { error -> handleError(error) /**🔥Crash🔥**/ } 
    )
``` 

我们可以想到使用上文提到的方法来解决这个问题

```
Single.just(getSomeData() )
   .map { item -> handleMap(item) }
   .doOnSuccess { result -> handleResult(result) }, }
   .doOnError { error -> handleError(error) /*🔥Crash🔥*/ }
   .subscribe({} {}) // Crash stop here ✅
```

尽管这样可以避免 crash ，但是仍然很奇怪，因为没有在 crash 时做任何事情，我们可以按照下面的方式，在 onError 中捕获异常，这是一种非常有趣的编程方式。

```
Single.just(getSomeData() )
   .map { item -> handleMap(item) }
   .doOnSuccess { result -> handleResult(result) }, }
   .doOnError { error -> handleError(error) /*🔥Crash🔥*/ }
   .subscribe({} { error -> handleError(error) }) // Crash caught ✅
```

不管怎么样这个方案是可行的，在这里只是展示如何处理，后面还会有很好的方式

### Crashes 发生在 complete 中

例如 Observable 除了 onError 和 onNext（还有类似于 Single 的 onSuccess）之外，还有onComplete 状态。

如果 crashes 发生在如下所示的 onComplete 中，它将不会被捕获。

```
Observable.just(getSomeData() )
   .map { item -> handleMap(item) } 
   .subscribe(
        { result -> handleResult(result) }, 
        { error -> handleError(error) }, // Crash NOT caught ⛔️
        { handleCompletion()/**🔥Crash🔥**/ }
    )
```

我们可以按照之前的方法在 doOnComplete 方法中进行处理，如下所示：

```
Observable.just(getSomeData() )
   .map { item -> handleMap(item) } 
   .doOnNext{ result -> handleResult(result) }
   .doOnError{ error -> handleError(error) } Crash NOT stop ⛔️
   .doOnComplete { handleCompletion()/**🔥Crash🔥**/ }
   .subscribe({ }, { }, { }) // Crash STOP here ✅
```

最终 crash 能够在 doOnError 中捕获，并在我们提供的最后一个空 onError 函数处停止，但是我们通过这种解决方法逃避了问题。

### Crashes 发生在 complete 之前

让我们看另一种有趣的情况，我们模拟一种情况，我们订阅的操作非常慢，无法轻易终止（如果终止，它将crash）

```
val disposeMe  = Observable.fromCallable { Thread.sleep(1000) }
    .doOnError{ error -> handleError(error) } // Crash NOT caught ⛔️
    .subscribe({}, {}, {}) // Crash NOT caught or stop ⛔️
Handler().postDelayed({ disposeMe.dispose() }, 100)
```

我们在 fromCallable 中等待 1000 才能完成，但是在100毫秒，我们通过调用 disposeMe.dispose() 终止操作。

这将迫使 Thread.sleep（1000）在结束之前终止，从而使其崩溃，无法通过 doOnError 或者提供的 onError 函数捕获崩溃

即使我们在最外面使用 try-catch，也是没用的，也无法像其他所有 RxJava 内部 crash 一样起作用。

```
try {
    val disposeMe  = Observable.fromCallable { Thread.sleep(1000) }
        .doOnError{} // Crash NOT caught ⛔️
        .subscribe({}, {}, {}) // Crash NOT caught or stop ⛔️
    Handler().postDelayed({ disposeMe.dispose() }, 100)
} catch (exception: Exception) {
    handleError(exception) // Crash NOT caught too ⛔️
}
```

###  RxJava Crash 终极解决方案

对于 RxJava 如果确实发生了 crash，但 crash 不在您的控制范围内，并且您希望采用一种全局的方式捕获它，可以用下面是解决方案。


```
RxJavaPlugins.setErrorHandler { e -> handleError(e) }
```

注册 ErrorHandler 它将捕获上述任何情况下的所有 RxJava 未捕获的错误（ just() 除外，因为它不属于RxJava 调用链的一部分）

但是要注意用于调用错误处理的线程在 crash 发生的地方挂起，如果你想确保它总是发生在主UI线程上，用 runOnUiThread{ } 包括起来

```
RxJavaPlugins.setErrorHandler { e -> 
    runOnUiThread { handleError(e))}
}
```

因此，对于上面的情况，由于在完成之前终止而导致 Crash，下面将对此进行处理。

```
RxJavaPlugins.setErrorHandler { e -> handle(e) } // Crash caught ✅
val disposeMe  = Observable.fromCallable { Thread.sleep(1000) }
    .doOnError{ error -> handleError(error) } // Crash NOT caught ⛔️
    .subscribe({}, {}, {}) // Crash NOT caught or stop ⛔️
Handler().postDelayed({ disposeMe.dispose() }, 100)
```

有了这个解决方案，并不意味着注册 ErrorHandler 就是正确的方式

```
RxJavaPlugins.setErrorHandler { e -> handle(e) }
```

通过了解上面发生 Crash 处理方案，您就可以选择最有效的解决方案，多个方案配合一起使用，以更健壮地处理程序所发生的的 Crash。


## 译者思考

作者大概总了 5 种 RxJava 可能出现的异常的位置

* Crashes 发生在 just() 中
* Crashes 发生在 subscribe success 中
* Crashes 发生在 subscribe error 中
* Crashes 发生在 complete 中
* Crashes 发生在 complete 之前

总的来说 RxJava 无法判断这些超出生命周期的、不可交付的异常中哪些应该或不应该导致应用程序崩溃，最后作者给出了，RxJava Crash 终极解决方案，注册 ErrorHandler

```
RxJavaPlugins.setErrorHandler { e -> handleError(e) }
```

它将捕获上述任何情况下的所有 RxJava 未捕获的错误，just() 除外，接下来我们来了解一下 RxJavaPlugins.setErrorHandler

### 关于 RxJavaPlugins.setErrorHandler

这是 RxJava2.x 的一个重要设计，以下几种类型的错误 RxJava 是无法捕获的：

* 由于下游的生命周期已经达到其终端状态导致的异常
* 下游取消了将要发出错误的序列而无法发出的错误
* 发生了 crash ，但是 crash 不在您的控制范围内
* 一些第三方库的代码在被取消 或者 调用中断时会抛出该异常，这通常会导致无法交付的异常。

RxJava 无法判断这些超出生命周期的、不可交付的异常中哪些应该或不应该导致应用程序崩溃。

这些无法捕获的错误，最后会发送到 RxJavaPlugins.onError 处理程序中。这个处理程序可以用方法RxJavaPlugins.setErrorHandler() 重写，RxJava 默认情况下会将 Throwable 的 stacktrace 打印到控制台，并调用当前线程的未捕获异常处理程序。

所以我们可以采用一种全局的处理方式，注册一个 RxJavaPlugins.setErrorHandler() ，添加一个非空的全局错误处理，下面的示例演示了上面列出来的几种无法交付的异常。

```
RxJavaPlugins.setErrorHandler(e -> {
    if (e instanceof UndeliverableException) {
        e = e.getCause();
    }
    if ((e instanceof IOException) || (e instanceof SocketException)) {
        // fine, irrelevant network problem or API that throws on cancellation
        return;
    }
    if (e instanceof InterruptedException) {
        // fine, some blocking code was interrupted by a dispose call
        return;
    }
    if ((e instanceof NullPointerException) || (e instanceof IllegalArgumentException)) {
        // that's likely a bug in the application
        Thread.currentThread().getUncaughtExceptionHandler()
            .handleException(Thread.currentThread(), e);
        return;
    }
    if (e instanceof IllegalStateException) {
        // that's a bug in RxJava or in a custom operator
        Thread.currentThread().getUncaughtExceptionHandler()
            .handleException(Thread.currentThread(), e);
        return;
    }
    Log.warning("Undeliverable exception received, not sure what to do", e);
});
```

我相信到这里关于 RxJava Crash 处理方案，应该了解的很清楚了，选择最有效的解决方案，多个方案配合一起使用，可以更健壮地处理程序所发生的的 Crash。

接下来我们来了解一下 just 和 fromCallable 区别 ，作者在另外一篇文章 [just-vs-fromcallable](https://medium.com/@elye.project/rxjava-2-understanding-hot-vs-cold-with-just-vs-fromcallable-3c463f9f68c9) 做了详细的介绍

### just 和 fromCallable 区别

#### 1. 值获取来源不一样

just 值从外部获取的，而 fromCallable 值来自于内部生成，为了更清楚的了解，我们来看一下下面的代码：

```
println("From Just")
val justSingle = Single.just(Random.nextInt())
justSingle.subscribe{ it -> println(it) }
justSingle.subscribe{ it -> println(it) }

println("\nFrom Callable")
val callableSingle = Single.fromCallable { Random.nextInt() }
callableSingle.subscribe{ it -> println(it) }
callableSingle.subscribe{ it -> println(it) }
```

对于 Just 和 fromCallable 分别调用 2 次 subscribe 执行结果如下所示：

```
From Just
801570614
801570614

From Callable
1251601849
2033593269
```

你会发现对于 just 无论 subscribe 多少次，生成的随机值都保持不变，因为该值是从 Observable 外部生成的，而 Observable 只是将其存储供以后使用。

但是对于 fromCallable 它是从 Observable 内部生成的，每次 subscribe 都会都会生成一个新的随机数。

#### 2. 立即执行和延迟执行

* just 在调用 subscribe 方法之前值已经生成了，属于立即执行。
* 而 fromCallable 是调用 subscribe 方法之后执行的，属于延迟执行。

为了更清楚的了解，我们来看一下下面的代码：

```
fun main() {
    println("From Just")
    val justSingle = Single.just(getRandomMessage())
    println("start subscribing")
    justSingle.subscribe{ it -> println(it) }
   
    println("\nFrom Callable")
    val callableSingle = Single.fromCallable { getRandomMessage() }
    println("start subscribing")
    callableSingle.subscribe{ it -> println(it) }
}

fun getRandomMessage(): Int {
    println("-Generating-")
    return Random.nextInt()
}
```

结果如下所示：

```
From Just
-Generating-
start subscribing
1778320787

From Callable
start subscribing
-Generating-
1729786515
```

对于 just 在调用 subscribe 之前打印了 -Generating-，而 fromCallable 是在调用 subscribe 之后才打印 -Generating-。

到这里文章就结束了，我们来一起探讨一个问题， 在 Java 时代 RxJava 确实帮助我们解决了很多问题，但是相对而言不好的地方 RxJava 里面各种操作符，理解起来确实很费劲，随着 Google 将 Kotlin 作为 Android 首选语言，那么 RxKotlin，有能给我们带来哪些好处了，在 Kotlin 时代，还有必要使用 RxKotlin 吗？

## 参考文献

* [Beyond Basic RxJava Error Handling](https://proandroiddev.com/beyond-basic-rxjava-error-handling-b0875d3877e0)
* [RxJava 2 : Understanding Hot vs. Cold with just vs. fromCallable](https://medium.com/@elye.project/rxjava-2-understanding-hot-vs-cold-with-just-vs-fromcallable-3c463f9f68c9)
* [What's different in 2.0](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0)

## 结语

致力于分享一系列 Android 系统源码、逆向分析、算法、翻译相关的文章，目前正在翻译一系列欧美精选文章，请持续关注，除了翻译还有对每篇欧美文章思考，如果对你有帮助，请帮我点个赞，感谢！！！期待与你一起成长。


