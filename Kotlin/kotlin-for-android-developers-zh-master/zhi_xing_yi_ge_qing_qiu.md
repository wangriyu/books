# 执行一个请求

对于感受我们要实现的想法而言，我们目前的文本是很好开始，但是现在是时候去请求一些显示在RecyclerView上的真正的数据了。我们将会使用[OpenWeatherMap] API来获取数据，还有一些普通类来现实这个请求。多亏Kotlin非常强大的互操作性，你可以使用任何你想使用的库，比如用[Retrofit]来执行服务器请求。当只是执行一个简单的API请求，我们可以不使用任何第三方库来简单地实现。

而且，如你所见，Kotlin提供了一些扩展函数来让请求变得更简单。首先，我们要创建一个新的Request类：

```kotlin
public class Request(val url: String) {
    public fun run() {
        val forecastJsonStr = URL(url).readText()
        Log.d(javaClass.simpleName, forecastJsonStr)
    }
}
```

我们的请求很简单地接收一个url，然后读取结果并在logcat上打印json。实现非常简单，因为我们使用`readText`，这是Kotlin标准库中的扩展函数。这个方法不推荐结果很大的响应，但是在我们这个例子中已经足够好了。

如果你用这些代码去比较Java，你会发现我们仅使用标准库就节省了大量的代码。比如`HttpURLConnection`、`BufferedReader`和需要达到相同效果所必要的迭代结果，管理连接状态、reader等部分的代码。很明显，这些就是场景背后函数所作的事情，但是我们却不用关心。

为了可以执行请求，App必须要有Internet权限。所以需要在`AndroidManifest.xml`中添加：：

```xml
<uses-permission android:name="android.permission.INTERNET" />
```


[OpenWeatherMap]: http://openweathermap.org/
[Retrofit]: https://github.com/square/retrofit