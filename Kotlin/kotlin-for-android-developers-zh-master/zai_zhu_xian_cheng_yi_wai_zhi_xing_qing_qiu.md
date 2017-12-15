# 在主线程以外执行请求

如你所知，HTTP请求不被允许在主线程中执行，否则它会抛出异常。这是因为阻塞住UI线程是一个非常差的体验。Android中通用的做法是使用`AsyncTask`，但是这些类是非常丑陋的，并且使用它们无任何副作用地实现功能也是非常困难的。如果你使用不小心，`AsyncTasks`会非常危险，因为当运行到`postExecute`时，如果Activity已经被销毁了，这里就会崩溃。

Anko提供了非常简单的DSL来处理异步任务，它满足大部分的需求。它提供了一个基本的`async`函数用于在其它线程执行代码，也可以选择通过调用`uiThread`的方式回到主线程。在子线程中执行请求如下这么简单：

```kotlin
async() {
    Request(url).run()
    uiThread { longToast("Request performed") }
}
```

`UIThread`有一个很不错的一点就是可以依赖于调用者。如果它是被一个`Activity`调用的，那么如果`activity.isFinishing()`返回`true`，则`uiThread`不会执行，这样就不会在Activity销毁的时候遇到崩溃的情况了。

假如你想使用`Future`来工作，`async`返回一个Java `Future`。而且如果你需要一个返回结果的`Future`，你可以使用`asyncResult`。

真的很简单，对吧？而且比`AsyncTasks`更加具有可读性。现在，我仅仅给请求发送了一个url，来测试我们是否可以正确接收内容，这样我们才能在Activity中把它画出来。我很快会讲到怎么去进行json解析和转换成app中的数据类，但是在我们继续之前，学习什么是数据类也是很重要的。

检查代码并审查url请求和包结构的代码。你可以运行app并且确保你可以在打印的json日志和请求完毕之后的toast。