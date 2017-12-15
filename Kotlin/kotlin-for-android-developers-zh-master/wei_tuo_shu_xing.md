# 委托属性

我们可能需要一个属性具有一些相同的行为，使用`lazy`或者`observable`可以被很有趣地实现重用。而不是一次又一次地去声明那些相同的代码，Kotlin提供了一个委托属性到一个类的方法。这就是我们知道的`委托属性`。

当我们使用属性的`get`或者`set`的时候，属性委托的`getValue`和`setValue`就会被调用。

属性委托的结构如下：

```kotlin
class Delegate<T> : ReadWriteProperty<Any?, T> {
	fun getValue(thisRef: Any?, property: KProperty<*>): T {
		return ...
	}
	
	fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
		...
	}
}
```

这个T是委托属性的类型。`getValue`函数接收一个类的引用和一个属性的元数据。`setValue`函数又接收了一个被设置的值。如果这个属性是不可修改（val），就会只有一个`getValue`函数。

下面展示属性委托是怎么设置的：

```kotlin
class Example {
	var p: String by Delegate()
}
```

它使用了`by`这个关键字来指定一个委托对象。