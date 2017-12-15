# 基础

举个例子，我们可以创建一个指定泛型类：

```kotlin
class TypedClass<T>(parameter: T) {
    val value: T = parameter
}
```

这个类现在可以使用任何的类型初始化，并且参数也会使用定义的类型，我们可以这么做：

```kotlin
val t1 = TypedClass<String>("Hello World!")
val t2 = TypedClass<Int>(25)
```

但是Kotlin很简单并且缩减了模版代码，所以如果编译器能够推断参数的类型，我们甚至也就不需要去指定它：

```kotlin
val t1 = TypedClass("Hello World!")
val t2 = TypedClass(25)
val t3 = TypedClass<String?>(null)
```

如第三个对象接收一个null引用，那仍然还是需要指定它的类型，因为它不能去推断出来。

我们可以像Java中那样在定义中指定的方式来增加类型限制。比如，如果我们想限制上一个类中为非null类型，我们只需要这么做：

```kotlin
class TypedClass<T : Any>(parameter: T) { 
	val value: T = parameter
}
```

如果你再去编译前面的代码，你将看到`t3`现在会抛出一个错误。可null类型不再被允许了。但是限制明显可以更加严厉。如果我们只希望`Context`的子类该怎么做？很简单：

```kotlin
class TypedClass<T : Context>(parameter: T) { 
	val value: T = parameter
}
```

现在所有继承`Context`的类都可以在我们这个类中使用。其它的类型是不被允许的。

当然，可以使用函数中。我们可以相当简单地构建泛型函数：

```kotlin
fun <T> typedFunction(item: T): List<T> {
	...
}
```