# 泛型例子

理论之后，我们转移到一些实际功能上面，这会让我们更加简单地掌握它。为了不重复发明轮子，我使用三个Kotlin标准库中的三个函数。这些函数让我们仅使用泛型的实现就可以做一些很棒的事情。它可以鼓舞你创建自己的函数。

#### let

`let`实在是一个简单的函数，它可以被任何对象调用。它接收一个函数（接收一个对象，返回函数结果）作为参数，作为参数的函数返回的结果作为整个函数的返回值。它在处理可null对象的时候是非常有用的，下面是它的定义：

```kotlin
inline fun <T, R> T.let(f: (T) -> R): R = f(this)
```

它使用了两个泛型类型：`T` 和 `R`。第一个是被调用者定义的，它的类型被函数接收到。第二个是函数的返回值类型。

我们怎么去使用它呢？你可能还记得当我们从数据源中获取数据时，结果可能是null。如果不是null，则把结果映射到`domain model`并返回结果，否则直接返回null：

```kotlin
if (forecast != null) dataMapper.convertDayToDomain(forecast) else null
```

这代码是非常丑陋的，我们不需要使用这种方式去处理可null对象。实际上如果我们使用`let`，都不需要`if`：

```kotlin
forecast?.let { dataMapper.convertDayToDomain(it) }
```

多亏`?.`操作符，`let`函数只会在`forecast`不是null的时候才会执行。否则它会返回null。也就是我们想达到的效果。

#### with

本书中我们大量讲了这个函数。`with`接收一个对象和一个函数，这个函数会作为这个对象的扩展函数执行。这表示我们根据推断可以在函数内使用`this`。

```kotlin
inline fun <T, R> with(receiver: T, f: T.() -> R): R = receiver.f()
```

泛型在这里也是以相同的方式运行：`T`代表接收类型，`R`代表结果。如你所见，函数通过`f: T.() -> R`声明被定义成了扩展函数。这就是为什么我们可以调用`receiver.f()`。

通过这个app，我们有几个例子：

```kotlin
fun convertFromDomain(forecast: ForecastList) = with(forecast) {
    val daily = dailyForecast map { convertDayFromDomain(id, it) }
    CityForecast(id, city, country, daily)
}
```

#### apply

它看起来于`with`很相似，但是是有点不同之处。`apply`可以避免创建builder的方式来使用，因为对象调用的函数可以根据自己的需要来初始化自己，然后`apply`函数会返回它同一个对象：

```kotlin
inline fun <T> T.apply(f: T.() -> Unit): T { f(); return this }
```

这里我们只需要一个泛型类型，因为调用这个函数的对象也就是这个函数返回的对象。一个不错的例子：

```kotlin
val textView = TextView(context).apply {
    text = "Hello"
    hint = "Hint"
    textColor = android.R.color.white
}
```

它创建了一个`TextView`，修改了一些属性，然后赋值给一个变量。一切都很简单，具有可读性和坚固的语法。让我们用在当前的代码中。在`ToolbarManager`中，我们使用这种方式来创建导航drawable：

```kotlin
private fun createUpDrawable() = with(DrawerArrowDrawable(toolbar.ctx)) {
    progress = 1f
	this
}
```

使用`with`和返回`this`是非常清晰的，但是使用`apply`可以更加简单：

```kotlin
private fun createUpDrawable() = DrawerArrowDrawable(toolbar.ctx).apply {
	progress = 1f
}
```

你可以在`Kotlin for Android Developer`代码库中查看这些小的优化。