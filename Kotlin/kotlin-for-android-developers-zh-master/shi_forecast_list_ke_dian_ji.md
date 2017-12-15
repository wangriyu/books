# 使Forecast list可点击

作为一个真正的app，当前列表的每一个item布局应该做一些工作。第一件事就是创建一个合适的XML，能符合我们的需要就行。我们希望显示一个图标，日期，描述以及最高和最低温度。所以让我们创建一个名为`item_forecast.xml`的layout：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:padding="@dimen/spacing_xlarge"
	android:background="?attr/selectableItemBackground"
	android:gravity="center_vertical"
	android:orientation="horizontal">
	
	<ImageView
		android:id="@+id/icon"
		android:layout_width="48dp"
		android:layout_height="48dp"
		tools:src="@mipmap/ic_launcher"/>
	
	<LinearLayout
		android:layout_width="0dp"
		android:layout_height="wrap_content"
		android:layout_weight="1"
		android:layout_marginLeft="@dimen/spacing_xlarge"
		android:layout_marginRight="@dimen/spacing_xlarge"
		android:orientation="vertical">
	
	<TextView
		android:id="@+id/date"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:textAppearance="@style/TextAppearance.AppCompat.Medium"
		tools:text="May 14, 2015"/>
	
	<TextView
		android:id="@+id/description"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:textAppearance="@style/TextAppearance.AppCompat.Caption"
		tools:text="Light Rain"/>
	
	</LinearLayout>
	<LinearLayout
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:gravity="center_horizontal"
		android:orientation="vertical">
	
	<TextView
		android:id="@+id/maxTemperature"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:textAppearance="@style/TextAppearance.AppCompat.Medium"
		tools:text="30"/>
	
	<TextView
		android:id="@+id/minTemperature"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:textAppearance="@style/TextAppearance.AppCompat.Caption"
		tools:text="15"/>
	
	</LinearLayout>
</LinearLayout>
```

Domain model和数据映射时必须生成完整的图标uil，所以我们可以这样去加载它：

```kotlin
data class Forecast(val date: String, val description: String,
					val high: Int, val low: Int, val iconUrl: String)
```

在`ForecastDataMapper`中：

```kotlin
private fun convertForecastItemToDomain(forecast: Forecast): ModelForecast {
    return ModelForecast(convertDate(forecast.dt),
            forecast.weather[0].description, forecast.temp.max.toInt(),
            forecast.temp.min.toInt(), generateIconUrl(forecast.weather[0].icon))
}

private fun generateIconUrl(iconCode: String): String
        = "http://openweathermap.org/img/w/$iconCode.png"
```

我们从第一个请求中得到图标的code，用来组成完成的图标url。加载图片最简单的方式是使用图片加载库。[`Picasso`]是一个不错的选择。它需要加到`build.gradle`的依赖中：

```groovy
compile "com.squareup.picasso:picasso:<version>"
```

如此，Adapter也需要一个大的改动了。还需要一个click listener，我们来定义它：

```kotlin
public interface OnItemClickListener {
     operator fun invoke(forecast: Forecast)
}
```

如果你还记得上一课程，当被调用时`invoke`方法可以被省略。所以我们来使用它来简化。listener可以被以下两种方式调用：

```kotlin
itemClick.invoke(forecast)
itemClick(forecast)
```

`ViewHolder`将负责去绑定数据到新的View：

```kotlin
class ViewHolder(view: View, val itemClick: OnItemClickListener) :
				RecyclerView.ViewHolder(view) {
    private val iconView: ImageView
    private val dateView: TextView
    private val descriptionView: TextView
    private val maxTemperatureView: TextView
    private val minTemperatureView: TextView

    init {
    	iconView = view.find(R.id.icon)
    	dateView = view.find(R.id.date)
    	descriptionView = view.find(R.id.description)
    	maxTemperatureView = view.find(R.id.maxTemperature)
    	minTemperatureView = view.find(R.id.minTemperature)
    }

    fun bindForecast(forecast: Forecast) {
    	with(forecast) {
    	    Picasso.with(itemView.ctx).load(iconUrl).into(iconView)
    	    dateView.text = date
    	    descriptionView.text = description
    	    maxTemperatureView.text = "${high.toString()}"
    	    minTemperatureView.text = "${low.toString()}"
    	    itemView.setOnClickListener { itemClick(forecast) }
	    }
    }
}
```

现在Adapter的构造方法接收一个`itemClick`。创建和绑定数据也是更简单：

```kotlin
public class ForecastListAdapter(val weekForecast: ForecastList,
         val itemClick: ForecastListAdapter.OnItemClickListener) :
        RecyclerView.Adapter<ForecastListAdapter.ViewHolder>() {
        
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int):
            ViewHolder {
        val view = LayoutInflater.from(parent.ctx)
            .inflate(R.layout.item_forecast, parent, false)
        return ViewHolder(view, itemClick)
    }
    
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bindForecast(weekForecast[position])
	}
	...
}
```

如果你使用了上面这些代码，`parent.ctx`不会被编译成功。Anko提供了大量的扩展函数来让Android编程更简单。举个例子，activitys、fragments以及其它包含了`ctx`这个属性，通过`ctx`这个属性来返回context，但是在View中缺少这个属性。所以我们要创建一个新的名叫`ViewExtensions.kt`文件来代替`ui.utils`，然后增加这个扩展属性：

```kotlin
val View.ctx: Context
    get() = context
```

从现在开始，任何View都可以使用这个属性了。这个不是必须的，因为你可以使用扩展的context属性，但是我觉得如果我们使用`ctx`的话在其它类中也会更有连贯性。而且，这是一个很好的怎么去使用扩展属性的例子。

最后，MainActivity调用setAdapter，最后结果是这样的：
```kotlin
forecastList.adapter = ForecastListAdapter(result,
        object : ForecastListAdapter.OnItemClickListener{
			override fun invoke(forecast: Forecast) {
			    toast(forecast.date)
		    }
	    })
```

如你所见，创建一个匿名内部类，我们去创建了一个实现了刚刚创建的接口的对象。看起来不是很好，对吧？这是因为我们还没开始试使用另一个强大的函数式编程的特性，但是你将会在下一章中学习到怎么去把这些代码转换得更简单。

去代码库中更新新的代码。UI开始看起来更好了。

[`Picasso`]: http://square.github.io/picasso/