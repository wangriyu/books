# 润色我们的代码

我们已经准备好使用`public`来进行重构了，但是我们还有很多其它细节需要修改。比如，在`RequestForecastCommand`中，我们在构造函数中我们创建的属性`zipCode`可以定义为`private`：

```kotlin
class RequestForecastCommand(private val zipCode: String)
```

所作的事情就是我们创建了一个不可修改的属性zipCode，它的值我们只能去得到，不能去修改它。所以这个不大的改动让代码看起来更加清晰。如果我们在编写类的时候，你觉得某些属性因为是什么原因不能对别人可见，那就把它定义为`private`。

而且，在Kotlin中，我们不需要去指定一个函数的返回值类型，它可以让编译器推断出来。举个省略返回值类型的例子：

```kotlin
data class ForecastList(...) {
	fun get(position: Int) = dailyForecast[position]
	fun size() = dailyForecast.size()
}
```

我们可以省略返回值类型的典型情景是当我们要给一个函数或者一个属性赋值的时候。而不需要去写代码块去实现。

剩下的修改是相当简单的，你可以在代码库中去同步下来。