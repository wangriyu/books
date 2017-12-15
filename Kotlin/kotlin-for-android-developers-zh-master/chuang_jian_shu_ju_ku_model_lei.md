# 创建数据库model类

但是首先，我们要去为数据库创建model类。你还记得我们之前所见的map委托的方式？我们要把这些属性直接映射到数据库中，反过来也一样。

我们先来看下`CityForecast`类：

```kotlin
class CityForecast(val map: MutableMap<String, Any?>,
                           val dailyForecast: List<DayForecast>) {
    var _id: Long by map
    var city: String by map
    var country: String by map
    
    constructor(id: Long, city: String, country: String,
                dailyForecast: List<DayForecast>)
    : this(HashMap(), dailyForecast) {
        this._id = id
        this.city = city
        this.country = country
    }
}
```

默认的构造函数会得到一个含有属性和对应的值的map，和一个dailyForecast。多亏了委托，这些值会根据key的名字会映射到相应的属性中去。如果我们希望映射的过程运行完美，那么属性的名字必须要和数据库中对应的名字一模一样。我们后面会讲原因。

但是，第二个构造函数也是必要的。这是因为我们需要从domain映射到数据库类中，所以不能使用map，从属性中设置值也是方便的。我们传入一个空的map，但是又一次，多亏了委托，当我们设置值到属性的时候，它会自动增加所有的值到map中。用这种方式，我们就准备好map来保存到数据库中了。使用了这些有用的代码，我将会看见它运行起来就像魔法一样神奇。

现在我们需要第二个类，DayForecast，它会是第二个表。它包括表中的每一列作为它的属性，它也有第二个构造函数。唯一不同之处就是不需要设置id，因为它将通过SQLite自增长。

```kotlin
class DayForecast(var map: MutableMap<String, Any?>) {
	var _id: Long by map
	var date: Long by map
	var description: String by map
	var high: Int by map
	var low: Int by map
	var iconUrl: String by map
	var cityId: Long by map
	
	constructor(date: Long, description: String, high: Int, low: Int,
				iconUrl: String, cityId: Long)
	: this(HashMap()) {
		this.date = date
		this.description = description
		this.high = high
		this.low = low
		this.iconUrl = iconUrl
		this.cityId = cityId
	}
}
```

这些类将会帮助我们SQLite表与对象之间的互相映射。