# 提供一个新的activity

现在我们准备去创建一个`DetailActivity`。我们详情activity将会接收一组从主activity传过来的参数：`forecast id`和`城市名称`。第一个参数将会用来从数据库中请求数据，城市名称用于显示在toolbar上。所以我们首先需要定义一组参数的名字：

```kotlin
public class DetailActivity : AppCompatActivity() {
	companion object {
	    val ID = "DetailActivity:id"
	    val CITY_NAME = "DetailActivity:cityName"
    }
    ...
}
```

在`onCreate`函数中，第一步是去设置content view。UI是非常简单的，但是对于这个app来说是足够了：

```kotlin
<LinearLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:orientation="vertical"
	android:paddingBottom="@dimen/activity_vertical_margin"
	android:paddingLeft="@dimen/activity_horizontal_margin"
	android:paddingRight="@dimen/activity_horizontal_margin"
	android:paddingTop="@dimen/activity_vertical_margin">
	
	<LinearLayout
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:orientation="horizontal"
		android:gravity="center_vertical"
		tools:ignore="UseCompoundDrawables">
	
		<ImageView
			android:id="@+id/icon"
			android:layout_width="64dp"
			android:layout_height="64dp"
			tools:src="@mipmap/ic_launcher"
			tools:ignore="ContentDescription"/>
		
		<TextView
			android:id="@+id/weatherDescription"
			android:layout_width="wrap_content"
			android:layout_height="wrap_content"
			android:layout_margin="@dimen/spacing_xlarge"
			android:textAppearance="@style/TextAppearance.AppCompat.Display1"
		    tools:text="Few clouds"/>
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <TextView
            android:id="@+id/maxTemperature"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:layout_margin="@dimen/spacing_xlarge"
            android:gravity="center_horizontal"
            android:textAppearance="@style/TextAppearance.AppCompat.Display3"
            tools:text="30"/>
        <TextView
            android:id="@+id/minTemperature"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:layout_margin="@dimen/spacing_xlarge"
            android:gravity="center_horizontal"
            android:textAppearance="@style/TextAppearance.AppCompat.Display3"
            tools:text="10"/>
    </LinearLayout>
</LinearLayout>
```

然后在`onCreate`代码中去设置它。使用城市的名字设置成toolbar的title。`intent`和`title`通过下面的方法被自动影射到属性：

```kotlin
setContentView(R.layout.activity_detail)
title = intent.getStringExtra(CITY_NAME)
```

`onCreate`实现的另一部分是调用command。这与我们之前做的非常相似：

```kotlin
async {
       val result = RequestDayForecastCommand(intent.getLongExtra(ID, -1)).execute()
       uiThread { bindForecast(result) }
}
```

当结果从数据库中获取之后，`bindForecast`函数在UI线程中被调用。我们在这个activity中又一次使用了Kotlin Android Extensions插件来实现不使用`findViewById`来从XML中获取到属性：

```kotlin
import kotlinx.android.synthetic.activity_detail.*
...

private fun bindForecast(forecast: Forecast) = with(forecast) {
       Picasso.with(ctx).load(iconUrl).into(icon)
       supportActionBar.subtitle = date.toDateString(DateFormat.FULL)
       weatherDescription.text = description
       bindWeather(high to maxTemperature, low to minTemperature)
}
```

这里有一些有趣的地方。比如，我创建了另一个扩展函数来转换一个`Long`对象到一个用于显示的日期字符串。记住我们在adapter中也使用了，所以明确定义它为一个函数是个不错的实践：

```kotlin
fun Long.toDateString(dateFormat: Int = DateFormat.MEDIUM): String {
    val df = DateFormat.getDateInstance(dateFormat, Locale.getDefault())
    return df.format(this)
}
```

我会得到一个`date format`（或者使用默认的DateFormat.MEDIUM）并转换`Long`为一个用户可以理解的`String`。

另一个有趣的地方是`bindWeather`函数。它会接收一个`vararg`的由`Int`和`TextView`组成的`pairs`，并且根据温度给`TextView`设置不同的`text`和`text color`。

```kotlin
private fun bindWeather(vararg views: Pair<Int, TextView>) = views.forEach {
    it.second.text = "${it.first.toString()}￿￿"
    it.second.textColor = color(when (it.first) {
        in -50..0 -> android.R.color.holo_red_dark
        in 0..15 -> android.R.color.holo_orange_dark
        else -> android.R.color.holo_green_dark
	})
}
```

每一个pair，它会设置一个`text`来显示温度和一个根据温度匹配的不同的颜色：低温度用红色，中温度用橙色，其它用绿色。温度值是比较随机的，但是这个是使用`when`表达式让代码变得简短精炼的很好的代表。

`color`是我想念的Anko中的一个扩展函数，它可以很简洁的方式从resources中获取一个color，类似于我们在其它地方使用到的`dimen`。我们写下这一行的时候，当前`support library`依赖`ContextCompat`来从不同的Android版本中获取一个color：

```kotlin
public fun Context.color(res: Int): Int = ContextCompat.getColor(this, res)
```

`AndroidManifest`也需要知道新activity的存在：

```kotlin
<activity
    android:name=".ui.activities.DetailActivity"
    android:parentActivityName=".ui.activities.MainActivity" >
    <meta-data
        android:name="android.support.PARENT_ACTIVITY"
        android:value="com.antonioleiva.weatherapp.ui.activities.MainActivity" />
</activity>
```