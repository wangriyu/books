# 访问Shared Preferences

你可能知道什么是Android [Shared Preferences]。可以通过Android框架简单存储的一系列key和value对。这些`preferences`与SDK的一部分融为一体，使得任务变得更加容易。而且从Android 6.0（Marshmallow），`shared preferences`可以自动被云存储，所以当一个用户在一个新的设备上面恢复App的时候，它们的`preferences`也会被恢复。

多亏使用了属性委托，我们可以使用非常简单的方式来处理`preferences`。我们可以创建一个委托，当`get`被调用时去查询，当`set`被调用时去执行保存操作。

因为我们想去保存`zip code`，它是一个long型，所以让我们创建一个Long属性的委托吧。在`DelegatesExtensions.kt`中，实现一个新的`LongPreference`类：

```kotlin
class LongPreference(val context: Context, val name: String, val default: Long)
    :  ReadWriteProperty<Any?, Long> {

	val prefs by lazy {
        context.getSharedPreferences("default", Context.MODE_PRIVATE)
    }

	override fun getValue(thisRef: Any?, property: KProperty<*>): Long {
        return prefs.getLong(name, default)
	}
	
	override fun setValue(thisRef: Any?, property: KProperty<*>, value: Long) {
        prefs.edit().putLong(name, value).apply()
    }
}
```

首先，我们使用`lazy`委托的方式创建一个preferences。这样的话，如果我们没有使用这个属性，这个委托就不会请求这个`SharedPreferences`对象。

当`get`被调用，它的实现是使用preferences实例去获取一个委托声明中指定名字的long属性，如果没有找到这个属性，则默认使用default。当一个值被`set`，拿到`preferences editor`并使用属性名保存。

我们可以在`DelegatesExt`中定义一个新的委托，这样我们访问时就简单很多：

```kotlin
object DelegatesExt {
    ....
    fun longPreference(context: Context, name: String, default: Long) =
        LongPreference(context, name, default)
}
```

在`SettingActivity`，现在可以定义一个属性去处理`zip code`偏好。我创建了两个常量用来作为名字和属性的默认值。这种方式可以在App其他地方使用：

```kotlin
companion object {
    val ZIP_CODE = "zipCode"
    val DEFAULT_ZIP = 94043L
}

var zipCode: Long by DelegatesExt.longPreference(this, ZIP_CODE, DEFAULT_ZIP)
```

现在preference工作起来就非常简单了，我们可以从属性中得到并赋值给`EditText`：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    ...
    cityCode.setText(zipCode.toString())
}
```

我们不能使用自动生成的属性`text`，因为`EditText`在`getText`中返回的是`Editable`，所以该属性默认为该值。如果我尝试去分配一个`String`，编译器会报错，使用`setText()`就足够了。

现在具备了所有要实现`onBackPressed`的东西。这里，一个属性的新值会被储存：

```kotlin
override fun onBackPressed() {
    super.onBackPressed()
    zipCode = cityCode.text.toString().toLong()
}
```

`MainActivity`需要一些小的改变。首先，它也需要一个`zip code`属性。

```kotlin
val zipCode: Long by DelegatesExt.longPreference(this, SettingsActivity.ZIP_CODE,
            SettingsActivity.DEFAULT_ZIP)
```

然后，我把`forecast load`的代码移动到了`onResume`，这样每次activity resumed，它都会刷新数据，以防`code zip`被修改。当然这里有更加复杂一点的方式去做，比如通过在请求forecast之前检查是否`zip code`真的改变了。但是我像保持这个例子的简单性，而且因为请求的数据已经保存在本地数据库中了，所以这个解决方案也不算太坏：

```kotlin
override fun onResume() {
    super.onResume()
    loadForecast()
}

private fun loadForecast() = async {
    val result = RequestForecastCommand(zipCode).execute()
    uiThread {
	    val adapter = ForecastListAdapter(result) {
		    startActivity<DetailActivity>(DetailActivity.ID to it.id,
		            DetailActivity.CITY_NAME to result.city)
		}
		forecastList.adapter = adapter
		toolbarTitle = "${result.city} (${result.country})"
	} 
}
```

`RequestForecastCommand`现在使用`zipCode`而不是之前的是一个固定值。

这里还有意见我们必须要做的事情：当溢出菜单的`settings`点击时启动这个`setting activity`。在`ToolbarManager`中的`initToolbar`函数需要有一些小的修改：

```kotlin
when (it.itemId) {
    R.id.action_settings -> toolbar.ctx.startActivity<SettingsActivity>()
    else -> App.instance.toast("Unknown option")
}
```

[Shared Preferences]: http://developer.android.com/training/basics/data-storage/shared-preferences.html