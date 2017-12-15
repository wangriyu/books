# 定义表

创建几个`objects`可以让我们避免表名列名拼写错误、重复等。我们需要两个表：一个用来保存城市的信息，另一个用来保存某天的天气预报。第二张表会有一个关联到第一张表的字段。

`CityForecastTable`提供了表的名字还有需要列：一个id（这个城市的zipCode），城市的名称和所在国家。

```kotlin
object CityForecastTable {
    val NAME = "CityForecast"
    val ID = "_id"
    val CITY = "city"
    val COUNTRY = "country"
}
```

`DayForecast`有更多的信息，就如你下面看到的有很多的列。最后一列`cityId`，用来保持属于的城市id。

```kotlin
object DayForecastTable {
	val NAME = "DayForecast"
	val ID = "_id"
	val DATE = "date"
	val DESCRIPTION = "description"
	val HIGH = "high"
	val LOW = "low"
	val ICON_URL = "iconUrl"
	val CITY_ID = "cityId"
}
```