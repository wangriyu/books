# ForecastListAdapter的click listener

在前面一章，我这么艰苦地写了click listener的目的就是更好的在这一章中进行开发。然而现在是时候把你学到的东西用到实践中去了。我们从ForecastListAdapter中删除了listener接口，然后使用lambda代替：

```kotlin
public class ForecastListAdapter(val weekForecast: ForecastList,
								 val itemClick: (Forecast) -> Unit)
```

这个itemClick函数接收一个`forecast`参数然后不返回任何东西。`ViewHolder`中也可以这么修改：

```kotlin
class ViewHolder(view: View, val itemClick: (Forecast) -> Unit)
```

其它的代码保持不变。仅仅改变`MainActivity`：

```kotlin
val adapter = ForecastListAdapter(result) { forecast -> toast(forecast.date) }
```

我们可以简化最后一句。如果这个函数只接收一个参数，那我们可以使用`it`引用，而不用去指定左边的参数。所以我们可以这么做：

```kotlin
val adapter = ForecastListAdapter(result) { toast(it.date) }
```