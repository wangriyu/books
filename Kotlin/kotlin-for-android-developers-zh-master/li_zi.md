# 例子

你可以想象，Kotlin List是实现了数组操作符的，所以我们可以像Java中的数组一样访问List的每一项。除此之外：在可修改的List中，每一项也可以用一个简单的方式被直接设置：

```kotlin
val x = myList[2]
myList[2] = 4
```

如果你还记得，我们有一个叫ForecastList的数据类，它是由很多其他额外的信息组成的。有趣的是可以直接访问它的每一项而不是请求内部的list得到某一项。做一个完全不相关的事情，我要去实现一个`size()`方法，它能稍微能简化一点当前的Adapter：

```kotlin
data class ForecastList(val city: String, val country: String,
                        val dailyForecast: List<Forecast>) {
    operator fun get(position: Int): Forecast = dailyForecast[position]
    fun size(): Int = dailyForecast.size
}
```

它会使我们的`onBindViewHolder`更简单一点：

```kotlin
override fun onBindViewHolder(holder: ViewHolder,
        position: Int) {
	with(weekForecast[position]) {
	    holder.textView.text = "$date - $description - $high/$low"
	}
}
```

当然还有`getItemCount()`方法：

```kotlin
override fun getItemCount(): Int = weekForecast.size()
```