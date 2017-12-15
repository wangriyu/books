# 密封（Sealed）类

密封类用来限制类的继承关系，这意味着密封类的子类数量是固定的。看起来就像是枚举那样，当你想在一个密封类的子类中寻找一个指定的类的时候，你可以事先知道所有的子类。不同之处在于枚举的实例是唯一的，而密封类可以有很多实例，它们可以有不同的状态。

我们可以实现，比如类似Scala中的`Option`类：这种类型可以防止null的使用，当对象包含一个值时返回`Some`类，当对象为空时则返回`None`：

```kotlin
sealed class Option<out T> {
    class Some<out T> : Option<T>()
    object None : Option<Nothing>()
}
```

有一件关于密封类很不错的事情是当我们使用`when`表达式时，我们可以匹配所有选项而不使用`else`分支：

```kotlin
val result = when (option) {
    is Option.Some<*> -> "Contains a value"
    is Option.None -> "Empty"
}
```


