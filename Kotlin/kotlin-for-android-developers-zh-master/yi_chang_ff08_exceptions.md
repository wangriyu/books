# 异常（Exceptions）

在Kotlin中，所有的`Exception`都是实现了`Throwable`，含有一个`message`且未经检查。这表示我们不会强迫我们在任何地方使用`try/catch`。这与Java中不太一样，比如在抛出`IOException`的方法，我们需要使用`try-catch`包围代码块。通过检查exception来处理显示并不是一个好的方法。像[Bruce Eckel]、[Rod Waldhoff]或[Anders Hejlsberg]等人可以给你关于这个更好的观点。

抛出异常的方式与Java很类似：

```kotlin
throw MyException("Exception message")
```

`try`表达式也是相同的：

```kotlin
try{
	// 一些代码
}
catch (e: SomeException) {
	// 处理
}
finally {
	// 可选的finally块
}
```

在Kotlin中，`throw`和`try`都是表达式，这意味着它们可以被赋值给一个变量。这个在处理一些边界问题的时候确实非常有用：

```kotlin
val s = when(x){
	is Int -> "Int instance"
	is String -> "String instance"
	else -> throw UnsupportedOperationException("Not valid type")
}
```

或者

```kotlin
val s = try { x as String } catch(e: ClassCastException) { null }
```

[Bruce Eckel]: http://www.mindview.net/Etc/Discussions/CheckedExceptions
[Rod Waldhoff]: http://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html
[Anders Hejlsberg]: http://www.artima.com/intv/handcuffs.html