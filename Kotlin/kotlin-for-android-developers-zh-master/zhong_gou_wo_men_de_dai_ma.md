# 重构我们的代码

现在是时候使用`Kotlin Android Extensions`来修改我们的代码了。修改相当简单。

我们从`MainActivity`开始。我们当前只是使用了`forecastList`的RecyclerView。但是我们可以简化一点代码。首先，为`activity_main`XML增加手工import：

```kotlin
import kotlinx.android.synthetic.activity_main.*
```

之前说过，我们使用id来访问views。所以我要修改`RecyclerView`的id，不使用下划线，让它更加适合Kotlin变量的名字。XML最后如下：

```xml
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <android.support.v7.widget.RecyclerView
        android:id="@+id/forecastList"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</FrameLayout>
```

然后现在，我们可以不需要`find`这一行了：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    forecastList.layoutManager = LinearLayoutManager(this)
    ...
}
```

这已经是最小的简化了，因为这个布局非常简单。但是`ForecastListAdapter`也可以从这个插件中受益。这里你可以使用一个装置来绑定这些属性到view中，它可以帮助我们移除所有`ViewHolder`的`find`代码。

首先，为`item_forecast`增加手工导入：

```kotlin
import kotlinx.android.synthetic.item_forecast.view.*
```

然后现在我们可以在`ViewHolder`中使用包含在`itemView`中的属性。实际上你可以在任何view中使用这些属性，但是很显然如果view不包含要获取的子view就会奔溃。

现在我们可以直接访问view的属性了：

```kotlin
class ViewHolder(view: View, val itemClick: (Forecast) -> Unit) :
        RecyclerView.ViewHolder(view) {
    fun bindForecast(forecast: Forecast) {
        with(forecast){
	        Picasso.with(itemView.ctx).load(iconUrl).into(itemView.icon)
			itemView.date.text = date
			itemView.description.text = description
			itemView.maxTemperature.text = "${high.toString()}￿￿"
			itemView.minTemperature.text = "${low.toString()}￿￿"
			itemView.onClick { itemClick(forecast) }
		} 
	}
}
```

Kotlin Android Extensions插件帮助我们减少了很多模版代码，并且简化了我们访问view的方式。从库中检出最新的代码吧。