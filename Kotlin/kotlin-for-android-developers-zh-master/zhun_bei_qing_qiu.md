# 准备请求

因为我们需要知道哪一个item我们要在详情界面中显示出来，所以逻辑告诉我们需要发送一个天气预报的`id`到详情界面。所以`domain model`需要一个新的`id`属性：

```kotlin
data class Forecast(val id: Long, val date: Long, val description: String,
    val high: Int, val low: Int, val iconUrl: String)
```

`ForecastProvider`也需要一个新的函数，它返回通过`id`请求后的结果。`DetailActivity`将需要通过接收到的`id`来执行请求获取天气预报数据。因为所有的请求都会迭代所有的数据源并且返回第一个非null的结果，我们可以抽取并定义一个新的函数：

```kotlin
private fun <T : Any> requestToSources(f: (ForecastDataSource) -> T?): T
        = sources.firstResult { f(it) }
```

这个函数使用一个非null类型作为范型。它会接收一个函数，并返回一个可null的对象。其中这个接收的函数接收一个`ForecastDataSource`，并返回一个可null范型的对象。我们可以重写上一个请求并如下写一个新的：

```kotlin
fun requestByZipCode(zipCode: Long, days: Int): ForecastList = requestToSources {
	val res = it.requestForecastByZipCode(zipCode, todayTimeSpan())
	if (res != null && res.size() >= days) res else null
}

fun requestForecast(id: Long): Forecast = requestToSources {
	it.requestDayForecast(id)
}
```

现在数据源需要去实现一个新的函数：

```kotlin
fun requestDayForecast(id: Long): Forecast?
```

`ForcastDb`将总是会拿到所需的在上一次请求被缓存的值，所以我们可以通过这种方式去获取它：

```kotlin
override fun requestDayForecast(id: Long): Forecast? = forecastDbHelper.use {
    val forecast = select(DayForecastTable.NAME).byId(id).
            parseOpt { DayForecast(HashMap(it)) }
    if (forecast != null) dataMapper.convertDayToDomain(forecast) else null
}
```

`select`从查询与之前的非常相似。我创建了另一个名为`byId`的工具函数，因为通过`id`来请求是很通用的，像这样使用一个函数可以简化处理过程也更具可读性。函数的实现也是相当简单：

```kotlin
fun SelectQueryBuilder.byId(id: Long): SelectQueryBuilder
        = whereSimple("_id = ?", id.toString())
```

它只是使用了`whereSimple`函数实现使用`_id`字段来查询数据。这个函数相当普通，但是如你所见，你可以根据你数据库结构的需要来创建需要的扩展函数，它可以大量地简化你代码的可读性。`DataMapper`有一些不值得一提的轻微改变。你可以通过代码库来查看它们。

另一方面，`ForecastServer`将不会再使用，因为信息总是会被缓存在数据库中。我们可以在一些奇怪的场景下实现一些代码保护，但是我们在这个例子中没有做任何处理，所以如果发生它也会只是抛出一个异常：

```kotlin
override fun requestDayForecast(id: Long): Forecast?
        = throw UnsupportedOperationException()
```

> `try`和`throw`是表达式
> > 在Kotlin中，几乎一切都是表达式，也就是说一切都会返回一个值。这在函数式编程中是非常重要的，当你使用`try-catch`处理边界的问题或者当抛出异常的时候。比如，在上一个例子中，我们可以给结果分配一个exception就算他们不是相同的类型，而不是必须要去创建一个完整的代码块。当我们需要在一个`when`分支中抛出一个exception的时候也是非常有用：
>>```kotlin
	val x = when(y){ 
				in 0..10 -> 1
                in 11..20 -> 2
                else -> throw Exception("Invalid")
	}
>>```
>> `try-catch`中也是一样，我们可以根据try的结果分配一个值：
>> ```kotlin
	val x = try{ doSomething() }catch{ null }
>> ```

最后一件我们需要做的事就是在新的activity中创建一个command来执行请求：

```kotlin
class RequestDayForecastCommand(
        val id: Long,
        val forecastProvider: ForecastProvider = ForecastProvider()) :
        Command<Forecast> {
    override fun execute() = forecastProvider.requestForecast(id)
}
```

请求返回一个将用于activity绘制UI的`Forecast`结果。