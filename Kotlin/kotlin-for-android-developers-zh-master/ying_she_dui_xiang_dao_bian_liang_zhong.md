# 映射对象到变量中

映射对象的每一个属性到一个变量中，这个过程就是我们知道的多声明。这就是为什么会有`componentX`函数被自动创建。使用上面的`Forecast`类举个例子：

```kotlin
val f1 = Forecast(Date(), 27.5f, "Shiny day")
val (date, temperature, details) = f1
```

上面这个多声明会被编译成下面的代码：

```kotlin
val date = f1.component1()
val temperature = f1.component2()
val details = f1.component3()
```

这个特性背后的逻辑是非常强大的，它可以在很多情况下帮助我们简化代码。举个例子，`Map`类含有一些扩展函数的实现，允许它在迭代时使用key和value：

```kotlin
for ((key, value) in map) {
	Log.d("map", "key:$key, value:$value")
}
```