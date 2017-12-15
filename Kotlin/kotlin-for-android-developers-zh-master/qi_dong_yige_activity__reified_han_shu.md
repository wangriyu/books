# 启动一个activity：reified函数

最后一步是从`main activity`启动一个`detail activity`。我们可以如下重写adapter实例：

```kotlin
val adapter = ForecastListAdapter(result) {
	val intent = Intent(MainActivity@this, javaClass<DetailActivity>())
	intent.putExtra(DetailActivity.ID, it.id)
	intent.putExtra(DetailActivity.CITY_NAME, result.city)
	startActivity(intent)
}
```

但是这是非常冗长的。一如既往地，Anko提供了简单得多的方式通过`reified function`来启动一个activity：

```kotlin
val adapter = ForecastListAdapter(result) {
    startActivity<DetailActivity>(DetailActivity.ID to it.id,
            DetailActivity.CITY_NAME to result.city)
}
```

`reified function`背后到底有什么魔法呢？就像你可能知道的那样，当我们在Java中创建一个范型函数，我们没有办法得到范型类型的Class。一个流行的变通方法是作为参数传入一个Class。在Kotlin中，一个内联（`inline`）函数可以被具体化（`reified`），这意味着我们可以在函数中得到并使用范型类型的Class。Anko真正使用它的一个简单的例子接下来会讲到（在这个例子中我只使用了`String`）：

```kotlin
public inline fun <reified T: Activity> Context.startActivity(
        vararg params: Pair<String, String>) {
    val intent = Intent(this, T::class.javaClass)
    params forEach { intent.putExtra(it.first, it.second) }
    startActivity(intent)
}
```

真正的实现要更加复杂一点因为它使用了一个很长的令人讨厌的`when`表达式来增加由类型决定的额外信息，但是在概念上来说它没有增加其它更有用的知识。

`Reified`函数是有一个可以简化代码和提高理解性的语法糖。在这个例子中，它通过获取到了范型类型的`javaClass`来创建了一个intent，迭代所有参数并增加到intent，然后使用Intent来启动activity。`reified`限制于activity的子类。

剩下的一点细节在代码库中已经说明。我们现在有一个非常简单（但是完整）的主从视图（`master-detail`）的App，它使用Kotlin实现，没有使用一行Java代码。