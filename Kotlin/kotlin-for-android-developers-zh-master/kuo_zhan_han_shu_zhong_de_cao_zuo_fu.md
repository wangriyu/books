# 扩展函数中的操作符

我们不需要去扩展我们自己的类，但是我需要去使用扩展函数扩展我们已经存在的类来让第三方的库能提供更多的操作。几个例子，我们可以去像访问List的方式去访问`ViewGroup`的view：

```kotlin
operator fun ViewGroup.get(position: Int): View = getChildAt(position)
```

现在真的可以非常简单地从一个`ViewGroup`中通过position得到一个view：

```kotlin
val container: ViewGroup = find(R.id.container)
val view = container[2]
```

别忘了去[Kotlin for Android developers repository]去查看这些代码。

[Kotlin for Android developers repository]:  https://github.com/antoniolg/Kotlin-for-Android-Developers