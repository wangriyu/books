# 扩展语言

多亏这些改变，我们可以去创建自己的`builder`和代码块。我们已经在使用一些有趣的函数，比如`with`。如下简单的实现：

```kotlin
inline fun <T> with(t: T, body: T.() -> Unit) { t.body() }
```

这个函数接收一个`T`类型的对象和一个被作为扩展函数的函数。它的实现仅仅是让这个对象去执行这个函数。因为第二个参数是一个函数，所以我们可以把它放在圆括号外面，所以我们可以创建一个代码块，在这这个代码块中我们可以使用`this`和直接访问所有的public的方法和属性：

```kotlin
with(forecast) {
	Picasso.with(itemView.ctx).load(iconUrl).into(iconView)
	dateView.text = date
	descriptionView.text = description
	maxTemperatureView.text = "$high"
	minTemperatureView.text = "$low"
	itemView.setOnClickListener { itemClick(this) }
}
```

> 内联函数
>> 内联函数与普通的函数有点不同。一个内联函数会在编译的时候被替换掉，而不是真正的方法调用。这在一些情况下可以减少内存分配和运行时开销。举个例子，如果我们有一个函数，只接收一个函数作为它的参数。如果是一个普通的函数，内部会创建一个含有那个函数的对象。另一方面，内联函数会把我们调用这个函数的地方替换掉，所以它不需要为此生成一个内部的对象。

另一个例子：我们可以创建代码块只提供`Lollipop`或者更高版本来执行：

```kotlin
inline fun supportsLollipop(code: () -> Unit) {
	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
		code()
	}
}
```

它只是检查版本，然后如果满足条件则去执行。现在我们可以这么做：

```kotlin
supportsLollipop {
	window.setStatusBarColor(Color.BLACK)
}
```

举个例子，Anko也是基于这个思想来实现`Android Layout`的`DSL`化。你也可以查看`Kotlin reference`中[`使用DSL来编写HTML`]的一个例子。

[`使用DSL来编写HTML`]: http://kotlinlang.org/docs/reference/type-safe-builders.html