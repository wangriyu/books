# 写入和查询数据库

`SqliteOpenHelper`只是一个工具，是SQL世界和OOP之间的一个通道。我们要新建几个类来请求已经保存在数据库中的数据，和保存新的数据。被定义的类会使用`ForecastDbHelper`和`DataMapper`来转换数据库中的数据到`domain models`。我仍旧使用默认值的方式来实现简单的依赖注入：

```kotlin
class ForecastDb(
    val forecastDbHelper: ForecastDbHelper = ForecastDbHelper.instance,
    val dataMapper: DbDataMapper = DbDataMapper()) {
    ...
}
```

所有的函数使用前面章节讲到过的`use()`函数。lambda返回的值也会被作为这个函数的返回值。所以让我们定义一个使用`zip code`和`date`来查询一个`forecast`的函数：

```kotlin
fun requestForecastByZipCode(zipCode: Long, date: Long) = forecastDbHelper.use {
	...
}
```

这么没有什么解释的：我们使用`use`函数返回的结果作为这个函数返回的结果。

#### 查询一个forecast

第一个要做的查询就是每日的天气预报，因为我们需要这个列表来创建一个`city`对象。Anko提供了一个简单的请求构建器，所以我们来利用下这个有利条件：

```kotlin
val dailyRequest = "${DayForecastTable.CITY_ID} = ? " +
    "AND ${DayForecastTable.DATE} >= ?"
    
val dailyForecast = select(DayForecastTable.NAME)
        .whereSimple(dailyRequest, zipCode.toString(), date.toString())
        .parseList { DayForecast(HashMap(it)) }
```

第一行，`dailyRequest`是查询语句中`where`的一部分。它是`whereSimple`函数需要的第一个参数，这与我们用一般的helper做的方式很相似。这里有另外一个简化的`where`函数，它需要一些tags和values来进行匹配。我不太喜欢这个方式，因为我觉得这个增加了代码的模版化，虽然这个对我们把values解析成String很有利。最后它看起来会是这样：

```kotlin
val dailyRequest = "${DayForecastTable.CITY_ID} = {id}" + "AND ${DayForecastTable.DATE} >= {date}"

val dailyForecast = select(DayForecastTable.NAME)
        .where(dailyRequest, "id" to zipCode, "date" to date)
        .parseList { DayForecast(HashMap(it)) }
```

你可以选择你喜欢的一种方式。`select`函数是很简单的，它仅仅是需要一个被查询表的名字。`parse`函数的时候会有一些魔法在里面。在这个例子中我们假设请求结果是一个list，使用了`parseList`函数。它使用了`RowParser`或`RapRowParser`函数去把cursor转换成一个对象的集合。这两个不同之处就是`RowParser`是依赖列的顺序的，而`MapRowParser`是从map中拿到作为column的key名的。

在它们之间有两个重载的冲突，所以我们不能直接使用简化的方式准确地创建需要的对象。但是没有什么是不能通过扩展函数来解决的。我创建了一个接收一个lambda函数返回一个`MapRowParser`的函数。解析器会调用这个lambda来创建这个对象：

```kotlin
fun <T : Any> SelectQueryBuilder.parseList(
    parser: (Map<String, Any>) -> T): List<T> =
        parseList(object : MapRowParser<T> {
            override fun parseRow(columns: Map<String, Any>): T = parser(columns)
})
```

这个函数可以帮助我们简单地去`parseList`查询的结果：

```kotlin
parseList { DayForecast(HashMap(it)) }
```

解析器接收的`immutable map`被我们转化成了一个`mutable map`（我们需要在`database model`中是可以修改的）通过使用相应的`HashMap`构造函数。在`DayForecast`中的构造函数中会使用到这个`HashMap`。

所以，这个查询返回了一个`Cursor`，要理解这个场景的背后到底发生了什么。`parseList`中会迭代它，然后得到`Cursor`的每一行直到最后一个。对于每一行，它会创建一个包含这列的key和给对应的key赋值后的map。然后把这个map返回给这个解析器。

如果查询没有任何结果，`parseList`会返回一个空的list。

下一步查询城市也是一样的方法：

```kotlin
val city = select(CityForecastTable.NAME)
        .whereSimple("${CityForecastTable.ID} = ?", zipCode.toString())
        .parseOpt { CityForecast(HashMap(it), dailyForecast) }
```

不同之处是：我们使用的是`parseOpt`。这个函数返回一个可null的对象。结果可以使一个null或者单个的对象，这取决于请求是否能在数据库中查询到数据。这里有另外一个叫`parseSingle`的函数，本质上是一样的，但是它返回的事一个不可null的对象。所以如果没有在数据库中找到这一条数据，它会抛出一个异常。在我们的例子中，第一次查询一个城市的时候，肯定是不存在的，所以使用`parseOpt`会更安全。我又创建了一个好用的函数来阻止我们需要的对象的创建：

```kotlin
public fun <T : Any> SelectQueryBuilder.parseOpt(
    parser: (Map<String, Any>) -> T): T? =
        parseOpt(object : MapRowParser<T> {
            override fun parseRow(columns: Map<String, Any>): T = parser(columns)
        })
```

最后如果返回的city不是null，我们使用`dataMapper`把它转换成`domain object`再返回它。否则，我们直接返回null。你应该记得，lambda的最后一行表示返回值。所以这里将会返回一个`CityForecast?`类型的对象：

```kotlin
if (city != null) dataMapper.convertToDomain(city) else null
```

`DataMapper`函数很简单：

```kotlin
fun convertToDomain(forecast: CityForecast) = with(forecast) {
    val daily = dailyForecast.map { convertDayToDomain(it) }
    ForecastList(_id, city, country, daily)
}

private fun convertDayToDomain(dayForecast: DayForecast) = with(dayForecast) {
	Forecast(date, description, high, low, iconUrl)
}
```

最后完整的函数如下：

```kotlin
fun requestForecastByZipCode(zipCode: Long, date: Long) = forecastDbHelper.use {

	val dailyRequest = "${DayForecastTable.CITY_ID} = ? AND " +
	    "${DayForecastTable.DATE} >= ?"
	val dailyForecast = select(DayForecastTable.NAME)
	        .whereSimple(dailyRequest, zipCode.toString(), date.toString())
	        .parseList { DayForecast(HashMap(it)) }
	        
	val city = select(CityForecastTable.NAME)
	        .whereSimple("${CityForecastTable.ID} = ?", zipCode.toString())
	        .parseOpt { CityForecast(HashMap(it), dailyForecast) }

    if (city != null) dataMapper.convertToDomain(city) else null
}
```

另外一个Anko中好玩的功能我们在这里展示，那就是你可以使用`classParser()`来替代我们用的`MapRowParser`，它是基于列名通过反射的方式去生成对象的。我喜欢另一种方法因为我不需要使用反射并且还有控制权进行转换操作，但是在有时候可能对你有用。

#### 保存一个forecast

`saveForecast`函数只是从数据库中清除数据，然后转换`domain` model为数据库model，然后插入每一天的`forecast`和`city forecast`。这个结构比之前的更简单：它通过`use`函数从`database helper`中返回数据。在这个例子中我们不需要返回值，所以它将返回`Unit`。

```kotlin
fun saveForecast(forecast: ForecastList) = forecastDbHelper.use {
    ...
}
```

首先，我们清空这两个表。Anko没有提供比较漂亮的方式来做这个，但这并不意味着我们不行。所以我们创建了一个`SQLiteDatabase`的扩展函数来让我们可以像SQL查询一样来执行它：

```kotlin
fun SQLiteDatabase.clear(tableName: String) {
	execSQL("delete from $tableName")
}
```

清空这两个表：

```kotlin
clear(CityForecastTable.NAME)
clear(DayForecastTable.NAME)
```

现在，是时候去转换执行`insert`后返回的数据了。在这一点上你可能直到我是`with`函数的粉丝：

```kotlin
with(dataMapper.convertFromDomain(forecast)) {
	...
}
```

从`domain model`转换的方式也是很直接的：

```kotlin
fun convertFromDomain(forecast: ForecastList) = with(forecast) {
    val daily = dailyForecast.map { convertDayFromDomain(id, it) }
    CityForecast(id, city, country, daily)
}

private fun convertDayFromDomain(cityId: Long, forecast: Forecast) =
    with(forecast) {
        DayForecast(date, description, high, low, iconUrl, cityId)
	}
```

在代码块，我们可以在不使用引用和变量的情况下使用`dailyForecast`和`map`，只是像我们在这个类内部一样就可以了。针对插入我们使用另外一个Anko函数，它需要一个表名和一个`vararg`修饰的`Pair<String, Any>`作为参数。这个函数会把`vararg`转换成Android SDK需要的`ContentValues`对象。所以我们的任务组成是把`map`转换成一个`vararg`数组。我们为`MutableMap`创建了一个扩展函数：

```kotlin
fun <K, V : Any> MutableMap<K, V?>.toVarargArray():
    Array<out Pair<K, V>> =  map({ Pair(it.key, it.value!!) }).toTypedArray()
```

它是支持可null的值的（这是`map delegate`的条件），把它转换为非null值（`select`函数需要）的`Array`所组成的`Pairs`。不用担心就算你不完全理解这个函数，我很快就会讲到可空性。

所以，这个新的函数我们可以这么使用：

```kotlin
insert(CityForecastTable.NAME, *map.toVarargArray())
```

它在`CityForecast`中插入了一个一行新的数据。在`toVarargArray`函数结果前面使用`*`表示这个array会被分解成为一个`vararg`参数。这个在Java中是自动处理的，但是我们需要在Kotlin中明确指明。

每天的天气预报也是一样了：

```kotlin
dailyForecast.forEach { insert(DayForecastTable.NAME, *it.map.toVarargArray()) }
```

所以，通过`map`的使用，我们可以用很简单的方式把类转换为数据表，反之亦然。因为我们已经新建了扩展函数，我们可以在别的项目中使用，这个才是真正可贵的地方。

这个函数的完整代码如下：

```kotlin
fun saveForecast(forecast: ForecastList) = forecastDbHelper.use {
	clear(CityForecastTable.NAME)
	clear(DayForecastTable.NAME)
	
	with(dataMapper.convertFromDomain(forecast)) {
	    insert(CityForecastTable.NAME, *map.toVarargArray())
	    dailyForecast forEach {
	        insert(DayForecastTable.NAME, *it.map.toVarargArray())
	    }
	}
}
```

在这一章中有很多代码被需要，所以你可以到代码库中查看检出。