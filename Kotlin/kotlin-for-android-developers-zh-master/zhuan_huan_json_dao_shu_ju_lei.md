# 转换json到数据类

我们现在知道怎么去创建一个数据类，那我们开始准备去解析数据。在`date`包中，创建一个名为`ResponseClasses.kt`新的文件，如果你打开第8章中的url，你可以看到json文件整个结构。它的基本组成包括一个城市，一个系列的天气预报，这个城市有id，名字，所在的坐标。每一个天气预报有很多信息，比如日期，不同的温度，和一个由描述和图标的id。

在我们当前的UI中，我们不会去使用所有的这些数据。我们会解析所有到类里面，因为可能会在以后某些情况下会用到。以下就是我们需要使用到的类：

```kotlin
data class ForecastResult(val city: City, val list: List<Forecast>)
data class City(val id: Long, val name: String, val coord: Coordinates,
                val country: String, val population: Int)
data class Coordinates(val lon: Float, val lat: Float)
data class Forecast(val dt: Long, val temp: Temperature, val pressure: Float,
                    val humidity: Int, val weather: List<Weather>,
                    val speed: Float, val deg: Int, val clouds: Int,
                    val rain: Float)
data class Temperature(val day: Float, val min: Float, val max: Float,
                       val night: Float, val eve: Float, val morn: Float)
data class Weather(val id: Long, val main: String, val description: String,
                   val icon: String)
```

当我们使用Gson来解析json到我们的类中，这些属性的名字必须要与json中的名字一样，或者可以指定一个`serialised name`（序列化名称）。一个好的实践是，大部分的软件结构中会根据我们app中布局来解耦成不同的模型。所以我喜欢使用声明简化这些类，因为我会在app其它部分使用它之前解析这些类。属性名称与json结果中的名字是完全一样的。

现在，为了返回被解析后的结果，我们的`Request`类需要进行一些修改。它将仍然只接收一个城市的`zipcode`作为参数而不是一个完整的url，因此这样变得更加具有可读性。现在，我会把这个静态的url放在一个`companion object`（伴随对象）中。如果我们之后还要对该API增加更多请求，我们需要提取它。

> __Companion objects__
>> Kotlin允许我们去定义一些行为与静态对象一样的对象。尽管这些对象可以用众所周知的模式来实现，比如容易实现的单例模式。

>> 我们需要一个类里面有一些静态的属性、常量或者函数，我们可以使用`companion objecvt`。这个对象被这个类的所有对象所共享，就像Java中的静态属性或者方法。

以下是最后的代码：

```kotlin
public class ForecastRequest(val zipCode: String) {
    companion object {
        private val APP_ID = "15646a06818f61f7b8d7823ca833e1ce"
        private val URL = "http://api.openweathermap.org/data/2.5/" +
                "forecast/daily?mode=json&units=metric&cnt=7"
        private val COMPLETE_URL = "$URL&APPID=$APP_ID&q="
	}
	
    fun execute(): ForecastResult {
        val forecastJsonStr = URL(COMPLETE_URL + zipCode).readText()
        return Gson().fromJson(forecastJsonStr, ForecastResult::class.java)
	}
}

```

记得在`build.gradle`中增加你需要的Gson依赖：

```groovy
compile "com.google.code.gson:gson:2.4"
```