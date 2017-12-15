# 创建业务逻辑来访问数据

在实现访问服务器和与本地数据库交互之后，是时候把事情整合起来了。逻辑步骤如下：

- 从数据库获取数据
- 检查是否存在对应星期的数据
- 如果有，返回UI并且渲染
- 如果没有，请求服务器获取数据
- 结果被保存在数据库中并且返回UI渲染

但是我们的`commands`不应该去处理所有这些逻辑。数据源应该是一个具体的实现，这样就可以被容易地修改，所以增加一些额外的代码，然后把`command`从数据访问中抽象出来听起来是个不错的方式。在我们的实现中，它会遍历整个list直到结果被找到。

所以我们先来给接口定义一些我们实现`provider`需要使用到的数据源：

```kotlin
interface ForecastDataSource {
    fun requestForecastByZipCode(zipCode: Long, date: Long): ForecastList?
}
```

`provider`需要一个接收`zip code`和一个`date`，然后它应该根据那一天返回一周的天气预报。

```kotlin
class ForecastProvider(val sources: List<ForecastDataSource> =
        ForecastProvider.SOURCES) {
        
	companion object {
	    val DAY_IN_MILLIS = 1000 * 60 * 60 * 24
	    val SOURCES = listOf(ForecastDb(), ForecastServer())
	}
    ...
}
```

`forecast provider`接收一个数据源列表，通过构造函数传入（比如用于测试），但是我设置了source的默认值为被定义在`companion object`中的`SOURCES`List。我将使用数据库的数据源和服务端数据源。顺序是很重要的，因为它会根据顺序去遍历这个sources，然后一旦获取到有效的返回值就会停止查询。逻辑顺序是先在本地查询（本地数据库中），然后再通过API查询。

所以主函数的代码如下：

```kotlin
fun requestByZipCode(zipCode: Long, days: Int): ForecastList
            = sources.firstResult { requestSource(it, days, zipCode) }
```

它会得到第一个不是null的结果然后返回。当我在第18章中讲到的大量的函数操作符中搜索后，我没有找到完全符合我想要的。所以当我去查看Kotlin的源码时，我直接拷贝了`first`函数然后修改它们来达到我想要的目的：

```kotlin
inline fun <T, R : Any> Iterable<T>.firstResult(predicate: (T) -> R?) : R {
	for (element in this){
		val result = predicate(element)
		if (result != null) return result
	}
	throw NoSuchElementException("No element matching predicate was found.")
}
```

该函数接收一个断言函数，它接收一个`T`类型的对象然后返回一个`R?`类型的值。这表示`predicate`可以返回null类型，但是我们的`firstResult`不能返回null。这就是为什么返回`R`的原因。

它怎么工作呢？它将遍历集合中的每一个元素然后执行这个断言函数。当这个断言函数的结果返回不是null时，这个结果就会被返回。

如果我们可以允许sources返回null，那我们就可以使用`firstOrNull`函数来代替。不同之处就是最后一行的返回null和抛异常。但是我现在不在代码里面去处理这些细节了。

在我们的例子中`T = ForecastDataSource`，`R = ForecastList`。但是记住在`ForecastDataSource`中指定的函数返回一个`ForecastList?`，也就是`R?`，所以一切都是匹配得这么完美。`requestSource`让前面的函数看起来更有可读性：

```kotlin
fun requestSource(source: ForecastDataSource, days: Int, zipCode: Long):
        ForecastList? {
    val res = source.requestForecastByZipCode(zipCode, todayTimeSpan())
    return if (res != null && res.size() >= days) res else null
}
```

如果结果不是null并且数量也参数匹配，那这个查询被执行且只会返回一个数据。否则，数据源没有足够的数据来返回一个成功的结果。

函数`todayTimeSpan()`计算今天毫秒级的时间，并排除掉“时差”。其中一些数据源（我们例子中的数据库）可能会需要它。因为如果我们没有指定更多的信息，服务端默认就是今天，所以我们不需要设置它。

```kotlin
private fun todayTimeSpan() = System.currentTimeMillis() / DAY_IN_MILLIS * DAY_IN_MILLIS
```

这个类完整的代码如下：

```kotlin
class ForecastProvider(val sources: List<ForecastDataSource> =
        ForecastProvider.SOURCES) {

	companion object {
        val DAY_IN_MILLIS = 1000 * 60 * 60 * 24;
        val SOURCES = listOf(ForecastDb(), ForecastServer())
    }

	fun requestByZipCode(zipCode: Long, days: Int): ForecastList
            = sources.firstResult { requestSource(it, days, zipCode) }

	private fun requestSource(source: RepositorySource, days: Int,
            zipCode: Long): ForecastList? {
        val res = source.requestForecastByZipCode(zipCode, todayTimeSpan())
        return if (res != null && res.size() >= days) res else null
    }

	private fun todayTimeSpan() = System.currentTimeMillis() /
            DAY_IN_MILLIS * DAY_IN_MILLIS
}
```

我们已经定义了一个`ForecastDb`。现在我们需要去实现`ForcastDataSource`：

```kotlin
class ForecastDb(val forecastDbHelper: ForecastDbHelper =
        ForecastDbHelper.instance, val dataMapper: DbDataMapper = DbDataMapper())
        : ForecastDataSource {
        
    override fun requestForecastByZipCode(zipCode: Long, date: Long) =
            forecastDbHelper.use {
            ...
	}
	...
}
```

`ForecastServer`还没有还被实现，但是这是非常简单的。它在从服务端接收到数据之后就会使用`ForecastDb`去保存到数据库。用这种方式，我们就可以缓存这些数据到数据库中，提供给以后的查询。

```kotlin
class ForecastServer(val dataMapper: ServerDataMapper = ServerDataMapper(),
        val forecastDb: ForecastDb = ForecastDb()) : ForecastDataSource {
        
    override fun requestForecastByZipCode(zipCode: Long, date: Long):
            ForecastList? {
        val result = ForecastByZipCodeRequest(zipCode).execute()
        val converted = dataMapper.convertToDomain(zipCode, result)
        forecastDb.saveForecast(converted)
        return forecastDb.requestForecastByZipCode(zipCode, date)
	}
}
```

它也是使用了之前我们创建的`data mapper`，最然我们修改一些函数的名字来让它更加与我们之前用在`database model`的mapper更相似。你可以查看`provider`来查看细节。

被重写的方法用来请求服务器，转换结果到`domain objects`并保存它们到数据库。它最后查询数据库返回数据，这是因为我们需要使用到插入到数据库中的字增长id。

这就是`provider`被实现的最后的一步了。现在我们需要开始使用它。`ForecastCommand`不会再直接与服务端交互，也不会转换数据到`domain model`。

```kotlin
RequestForecastCommand(val zipCode: Long,
        val forecastProvider: ForecastProvider = ForecastProvider()) :
        Command<ForecastList> {
        
	companion object {
		val DAYS = 7
	}
	
	override fun execute(): ForecastList {
		return forecastProvider.requestByZipCode(zipCode, DAYS)
	}
}
```

其它修改的地方包括重命名和包的结构调整。在[Kotlin for Android Developers repository]查看相应的提交。

[Kotlin for Android Developers repository]: https://github.com/antoniolg/Kotlin-for-Android-Developers