# 泛型preference委托

现在我们已经是泛型专家了，为什么不扩展`LongPreference`为支持所有`Shared Preferences`支持的类型呢？我们来创建一个`Preference`委托：

```kotlin
class Preference<T>(val context: Context, val name: String, val default: T)
    : ReadWriteProperty<Any?, T> {
	
	val prefs by lazy {
	    context.getSharedPreferences("default", Context.MODE_PRIVATE)
    }
	
	override fun getValue(thisRef: Any?, property: KProperty<*>): T {
		return findPreference(name, default)
	}

	override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        putPreference(name, value)
    }
	...
}
```

这个preference与我们之前使用的非常相似。我们仅仅替换了`Long`为泛型类型`T`，然后调用了两个函数来做具体重要的工作。这些函数非常简单，尽管有些重复。它们会检查类型然后使用指定的方式来操作。比如，`findPrefernce`函数如下：

```kotlin
private fun <T> findPreference(name: String, default: T): T = with(prefs) {
        val res: Any = when (default) {
        is Long -> getLong(name, default)
        is String -> getString(name, default)
        is Int -> getInt(name, default)
        is Boolean -> getBoolean(name, default)
        is Float -> getFloat(name, default)
        else -> throw IllegalArgumentException(
            "This type can be saved into Preferences")
    }

	res as T
}
```

`putPreference`函数也是一样，但是在`when`最后通过`apply`，使用`preferences editor`保存结果：

```kotlin
private fun <U> putPreference(name: String, value: U) = with(prefs.edit()) {
    when (value) {
        is Long -> putLong(name, value)
        is String -> putString(name, value)
        is Int -> putInt(name, value)
        is Boolean -> putBoolean(name, value)
        is Float -> putFloat(name, value)
        else -> throw IllegalArgumentException("This type can be saved into Pref\
erences")
    }.apply()
}
```

现在修改`DelegateExt`：

```kotlin
object DelegatesExt {
	...
	fun preference<T : Any>(context: Context, name: String, default: T)
	    = Preference(context, name, default)
}
```

这章之后，用户可以访问设置界面并修改`zip code`。然后当我们返回主界面，forecast会自动重新刷新并显示新的信息。查看代码库中其余xi wei
